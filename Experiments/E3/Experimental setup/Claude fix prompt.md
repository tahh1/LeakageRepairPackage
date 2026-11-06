For every notebook in Experiments/E3/Experimental setup/Fixed notebooks tracker.csv where assigned_to == "me" and status == "pending", spawn one subagent per notebook, in parallel where possible.

Give each subagent exactly this prompt, substituting the notebook, report and Dockerfile paths, and nothing else — no extra context, hints, or instructions about what to look for:

"You are reviewing the Jupyter notebook at {notebook_path}.

Find and fix any machine learning data leakage in this notebook.

A static analysis tool was previously run over this notebook. Its report is at {report_path}. Read it before you start.

The report is a JSON object whose single key, "Model pairs with data leakage", holds a list of model pairs. A model pair is one model-training call together with one evaluation call that uses the model it trained — a notebook that trains a model once and evaluates it twice yields two pairs. The tool analyzes each pair separately. Each entry has:

- "snippet": the program slice for that pair — the subset of the notebook's statements that affect the data flowing into its training and evaluation calls. It is not the whole notebook, and its statements are not necessarily adjacent in it. Each line is prefixed with its line number in the notebook, so the numbers are not contiguous.
- "model info": which call trains the model and which evaluates it, with their line numbers.
- "preproc_leakage_detected" / "overlap_leakage_detected": whether each kind was found for that pair.
- "preproc_leakage_lines" / "overlap_leakage_lines": the lines held responsible, each with an explanation.

Because the pairs are analyzed independently, one underlying problem affecting several pairs is reported separately under each of them, so the same line can appear under more than one pair and a single edit may resolve findings listed under several.

The notebook's environment is defined in the Dockerfile at {dockerfile_path}. You may build and run this Docker environment to execute the notebook as you work, if that helps you find or verify the fix.

## What the two kinds of leakage are

**Preprocessing leakage** occurs when a transformation that learns from data — scaling, normalization, encoding, imputation, feature selection, dimensionality reduction, vectorization — is fitted on data that includes the evaluation set, typically because it runs before the train/test split.

**Overlap leakage** occurs when samples, rows, windows, time steps or records used to train the model are reused to evaluate it.

## How to repair what you find

### Fit every stateful transformation on training data only

A transformation is **stateful** when it learns something from the data it is fitted on: scalers, imputers, encoders, vectorizers, feature selection, PCA and other decompositions, resamplers. Stateless row-wise operations — a log transform, arithmetic within a row, dropping a named column — carry no fitted state and may be applied before the split.

Relocating a stateful transformation after the split creates a case the original code never had to handle: the evaluation split may contain categories, tokens or levels absent from the training split. The repair must handle this deterministically, rather than raising at transform time or silently producing a different set of columns for the two splits.

For one-hot encoding, two implementations satisfy this:

- `OneHotEncoder(handle_unknown="ignore")`, fitted on the training split and applied to both;
- `pd.get_dummies` applied to each split separately, with the evaluation split reindexed to the training split's columns and missing columns zero-filled.

Prefer the encoder when the step has to sit inside a `Pipeline`, since `get_dummies` is not a transformer and cannot be a pipeline step.

`LabelEncoder` is a different case. It is designed for a one-dimensional target, has no mechanism for unseen labels, and raises when it meets one at transform time. On a feature column, replace it with `OrdinalEncoder(handle_unknown="use_encoded_value", unknown_value=-1)`. On the target it is appropriate and should be left alone.

Where the choice is otherwise free, preserve the original feature representation: if the original passed `drop_first=True`, use `drop="first"`; keep column names and ordering as close to the original as the API allows.

### Wrap stateful transformations in a `Pipeline` before passing them to any cross-validating call

`cross_val_score`, `cross_validate`, `GridSearchCV`, `RandomizedSearchCV` and the rest of that family make their own internal splits of whatever array you hand them. If a transformation was already fitted on that whole array, it has seen every validation fold the search will later create: each fold is then scored on data that influenced its own preprocessing. The resulting scores are optimistic, and because they drive model and hyperparameter selection, the bias is carried into the chosen configuration.

This is a second, independent leakage site. Moving a transformation to after the train/test split does not fix it — the transformation is still fitted once on all of the training data, while the search splits that same data again.

```python
# Leaky: the scaler has seen every validation fold before the search begins
scaler = StandardScaler()
X_train_s = scaler.fit_transform(X_train)
search = GridSearchCV(SVC(), {"C": [0.1, 1, 10]}, cv=5)
search.fit(X_train_s, y_train)

# Correct: the scaler is refit on the inner training fold at every split
pipe = Pipeline([("scale", StandardScaler()), ("model", SVC())])
search = GridSearchCV(pipe, {"model__C": [0.1, 1, 10]}, cv=5)
search.fit(X_train, y_train)
```

Two things to carry through the rewrite: parameter-grid keys must be renamed to the pipeline's step prefix (`C` becomes `model__C`), and resamplers such as SMOTE need `imblearn.pipeline.Pipeline` rather than sklearn's, so that resampling applies to the training fold only.

### Leave a flagged overlap unchanged when the evaluation data is semantically identical to the training data

A model evaluated on exactly the data it was trained on is an intentional evaluation of fit — an overfitting check — not a defect. The tool reports such pairs because reporting overlap at pair level is its job; deciding whether to act on a report is yours. Leave these pairs exactly as they are, including the reported training score. Removing the evaluation, or redirecting it to held-out data, destroys a result the notebook's author meant to present.

Two data references are **semantically identical** when, as determined from the notebook code, they refer to the same set of samples. Aliasing, reassignment, and order-preserving row-wise transformations preserve identity; adding, removing, resampling, or reordering rows does not.

```python
# Identical — leave unchanged
model.fit(X_train, y_train)
train_score = model.score(X_train, y_train)      # same reference

X_eval = X_train                                  # alias
train_pred = model.predict(X_eval)

X_train_s = scaler.transform(X_train)             # row-wise, order-preserving
train_pred = model.predict(X_train_s)

# Not identical — this is leakage, repair it
X_eval = X_train.sample(500)                      # subset
X_eval = pd.concat([X_train, extra_rows])         # superset
X_eval = resample(X_train, replace=True)          # resampled
X_eval = augment(X_train)                         # derived
```

The exemption is deliberately narrow. Evaluating on precisely the training set is a recognizable idiom with a single plausible reading, which is what licenses the inference that it was intended. A subset, a superset, or any resampled or derived variant supports no such inference: it may be an oversight as easily as a choice, so treat the overlap as leakage and correct it.

Apply the test per pair. A notebook may evaluate one model on its training data and another on a leaky subset; exempting the first says nothing about the second. When you exempt a pair, say so in your summary and state why, rather than passing over it silently.

### Anchor each repair to the split that feeds it

A notebook may contain more than one train/test split. Do not assume the first split in the file is the relevant one for every model. For each training call, trace backwards to identify which split's output variables feed it; that split is the anchor for repairing that model's leakage. Relocating a transformation to the wrong split will not remove the leakage and may introduce new errors.

Where several splits feed several models, treat each model independently. Do not apply one global relocation and assume it covers them all: scope each stateful transformation to the split it belongs to.

### Keep the change targeted

Do not add features, abstractions, refactors or validation beyond what the repair requires, and do not touch code that is not implicated. Never remove a transformation, an evaluation, or an output in order to make a finding go away — relocate or rewrite it instead. Report what you did faithfully; if you did not run a verification step, say so.

Make your edits directly in the notebook file. When you are done, summarize in a few sentences what you found and what you changed. If you found nothing, say so explicitly."

After each subagent finishes: save its full transcript to Experiments/E3/transcripts/{notebook_id}.log, update that row's subagent_summary in the tracker with its summary, and set status to completed.