# Where the E3 repair rules come from

The repair rules in the E3 fixing prompt come from reading the source code of
leaky notebooks that are not used in the experiments. For each rule, this
document shows one or two representative notebooks with the pattern behind it.

| Rule | Notebooks |
|---|---|
| 1. Fit every stateful transformation on training data only | `2021-09-14-nb_65`, `akmuthun_customer-churn-prediction`, `padmarajg_house-price-count-encoding-and-simple-imputer` |
| 2. Wrap stateful transformations in a `Pipeline` before any cross-validating call | `2021-09-05-nb_2770`, `2021-09-30-nb_3336` |
| 3. Leave a flagged overlap unchanged when the evaluation data is identical to the training data | `adithyabshetty100_coronary-heart-disease-prediction`, `akmuthun_customer-churn-prediction`, `nibukdk93_wine-quality-prediction` |
| 4. Anchor each repair to the split that feeds it | `padmarajg_house-price-count-encoding-and-simple-imputer`, `samuelkurabachew_titanic-predicting-survivors-beginners-notebook` |

These notebooks are not included in this package. They are available in the
annotated notebook corpus of the following studies:

- T. Draoui, M. W. Mkaouer, and C. D. Newman, "Improving data leakage
  detection in machine learning notebooks through static slicing and
  structured LLM prompts," *Proc. ACM Softw. Eng.*, vol. 3, no. FSE, 2026,
  doi: 10.1145/3808199.
- T. Draoui, Z. ElMabrouk, M. W. Mkaouer, M. Ben Messaoud, I. Jenhani,
  C. D. Newman, and H. Souari, "Understanding how data leakage manifests in
  real-world machine learning notebooks," *Inf. Softw. Technol.*, vol. 200,
  Art. no. 108336, 2026, doi: 10.1016/j.infsof.2026.108336.

Line numbers in the code excerpts refer to each notebook's Python-script
version.

---

## Rule 1: Fit every stateful transformation on training data only

The prompt asks the fixer to move encoders and other fitted transformations
after the split, and to handle categories that are unseen at transform time.
It names `OneHotEncoder(handle_unknown="ignore")` and per-split
`get_dummies` with reindexing as the acceptable repairs.

**`2021-09-14-nb_65`: `OneHotEncoder` fitted on the full dataset, then split.**

```python
39  ct = ColumnTransformer(transformers=[('encoder', OneHotEncoder(), [3])], remainder='passthrough')
40  x = np.array(ct.fit_transform(x))
...
55  x_train, x_test, y_train, y_test = train_test_split(x, y, test_size = 0.2, random_state = 0)
```

The encoder learns its category set from every row, including the rows that
later become `x_test`. Moving the `fit_transform` after line 55 is the
natural repair. Doing so is what raises the unseen-category question that the
rule's `handle_unknown="ignore"` clause answers.

**`akmuthun_customer-churn-prediction`: `get_dummies(drop_first=True)` and a
scaler on all rows, then split.**

```python
160  data_pre_processed = pd.get_dummies(data_no_mv_no_id,drop_first=True)
...
176  scaler.fit(inputs)
178  inputs_scaled = scaler.transform(inputs)
...
182  x_train, x_test, y_train, y_test = train_test_split(inputs_scaled, targets, test_size=0.2, random_state=1)
```

This is the `get_dummies` variant of the same pattern, with a fitted scaler
layered on top. The `drop_first=True` argument is why the rule asks the fixer
to preserve the original representation (`drop="first"`) when it rewrites the
encoding.

**`LabelEncoder` on feature columns.** In
`padmarajg_house-price-count-encoding-and-simple-imputer`, lines 503–505 loop
`LabelEncoder().fit_transform` over every categorical feature column of the
concatenated train+test frame. Moving this after a split would make
`LabelEncoder` raise on unseen labels. That is why the rule replaces it with
`OrdinalEncoder(handle_unknown="use_encoded_value", unknown_value=-1)` on
feature columns.

---

## Rule 2: Wrap stateful transformations in a `Pipeline` before any cross-validating call

The prompt treats `cross_val_score` and `GridSearchCV` as a second,
independent leakage site. It also states that moving a transformation after
the train/test split does not fix that site.

**`2021-09-05-nb_2770`: resampling and scaling on all rows, then CV and grid
search on the result.**

```python
274  sm=SMOTE()
275  x,y=sm.fit_resample(dfx,dfy)
...
304  x=power_transform(x,method=('yeo-johnson'))
...
313  sc=StandardScaler()
314  x=sc.fit_transform(x)
...
376  scores=cross_val_score(svc,x,y,cv=5)
...
411  clf=GridSearchCV(GradientBoostingClassifier(),parameters,cv=5,scoring='roc_auc')
412  clf.fit(x,y)
...
437  clf=GridSearchCV(SVC(),parameters,cv=5,scoring='roc_auc')
438  clf.fit(x,y)
```

Every validation fold that `cross_val_score` and `GridSearchCV` create has
already influenced the SMOTE neighbours and the scaler statistics. The
`best_params_` chosen at lines 412 and 438 are then used to configure the
final models (line 419). This notebook also motivates the clause on
resamplers: SMOTE has to sit in an `imblearn.pipeline.Pipeline` so that it
resamples only the training fold.

**`2021-09-30-nb_3336`: the scaler is already fitted after the split, but
the search splits `X_train` again.**

```python
177  X_train,X_test,y_train,y_test=train_test_split(X,y,test_size=0.25,random_state=42)
...
183  Sca=StandardScaler()
184  Sca.fit(X_train)
185  X_train=Sca.transform(X_train)
186  X_test=Sca.transform(X_test)
...
225  cross_val_score(sv,X_train,y_train,scoring='r2')
...
233  grid=GridSearchCV(sv1,param_grid=param_,cv=5,scoring='r2')
234  grid.fit(X_train,y_train)
```

This notebook already follows Rule 1, because the scaler sees only
`X_train`. The CV calls still leak, because each inner validation fold was
part of the data the scaler was fitted on. This is the case behind the
prompt's statement that "moving a transformation to after the train/test
split does not fix it". The same pattern repeats for the decision tree,
XGBoost, linear and ridge models later in the notebook (lines 331–332,
406–407, 438, 500 and 563).

---

## Rule 3: Leave a flagged overlap unchanged when the evaluation data is identical to the training data

The prompt exempts a model that is scored on exactly its own training set,
on the grounds that this is a deliberate check for overfitting. The pool
shows that authors use this idiom on purpose and say so in the code.

**`adithyabshetty100_coronary-heart-disease-prediction`: train and test
scores compared explicitly to diagnose overfitting.**

```python
945  acc_test = logreg.score(X_test, y_test)
946  print(("The accuracy score of the test data is: ",acc_test*100,"%"))
947  acc_train = logreg.score(X_train, y_train)
948  print(("The accuracy score of the training data is: ",round(acc_train*100,2),"%"))
...
953  # The scores for test and training data for the logistic regression model are similar. Therefore we do not expect the model to overfit
```

The training-set score at line 947 is the evidence for the author's
conclusion at line 953. Redirecting it to held-out data, or deleting it,
would remove the point the cell is making.

**`akmuthun_customer-churn-prediction`: a training-set evaluation that is
labelled as such, followed by the test evaluation.**

```python
193  log_model.fit(x_train,y_train)
194  # get prediction on train data itself to measure the performance of the model
195  y_hat = log_model.predict(x_train)
...
203  print(('acuracy score for training data = ', accuracy_score(y_hat,y_train)))
...
210  predictions = log_model.predict(x_test)
211  print(('confusion matrix for test data = ', confusion_matrix(predictions,y_test)))
```

The comment at line 194 states the intent directly, and the held-out
evaluation follows immediately after.

**Why the exemption is narrow.** The pool also contains overlap that is *not*
a deliberate check of fit. In `nibukdk93_wine-quality-prediction`, SMOTE runs
on the full feature set (line 270, `X_res, y_res = smote.fit_resample(X,y)`)
before the split (line 284). The test set therefore contains synthetic rows
interpolated from training rows. That is a derived variant of the training
data, not the training data itself, and nothing in the code suggests it was
intended. The rule's "subset, superset, resampled or derived" clause exists
to keep cases like this classified as leakage.

---

## Rule 4: Anchor each repair to the split that feeds it

The prompt tells the fixer to trace each model back to the split that
produces its training and evaluation data, rather than assuming the first
split is the relevant one. In both notebooks below, a fix anchored to the
first split looks correct but leaves the model's actual evaluation data
leaked.

**`padmarajg_house-price-count-encoding-and-simple-imputer`: an encoder fitted
"after the split", then that training set split again for evaluation.**

```python
460  train_test=pd.concat((train,test)).reset_index(drop=True)
...
475  imputed_train_test_num=pd.DataFrame(my_imputer.fit_transform(train_test_num))
477  imputed_train_test_obj = pd.DataFrame(my_obj_imputer.fit_transform(train_test_obj))
...
543  X_train = imputed_train_test.iloc[:len(y), :]       # first split: labelled rows
544  X_test = imputed_train_test.iloc[len(y):, :]        # vs. unlabelled submission rows
...
552  for col in set(train_test_obj):
553      target_enc = ce.TargetEncoder(cols=col)
554      target_enc.fit(X_train[col],y)
555      X_train[col+'_target' ]=target_enc.transform (X_train[col])
...
586  X_train, X_valid, y_train, y_valid = train_test_split(X_train, y, train_size=0.8, test_size=0.2,
587  random_state=0)
...
708  print((rmsle(y_valid, blend_models_predict(X_valid))))
```

The author's own reasoning (comments at lines 491 and 547) is that target
encoding becomes safe once the data is split at line 543. A fixer reasoning
the same way would move the imputers (lines 475–477) after line 543 and
treat the target encoder at line 554 as already correct. However, the only
labelled evaluation is on `X_valid` (line 708), which is carved out of
`X_train` at line 586. The target encoder, and any imputer re-anchored to
line 543, would still be fitted on the `X_valid` rows, including their target
values. The split that feeds the evaluated models is the one at line 586, so
that is where these transformations have to be anchored.

**`samuelkurabachew_titanic-predicting-survivors-beginners-notebook`:
imputation on the "training file", then a validation split of that file.**

```python
 90  train_data["Age"].fillna(train_data["Age"].median(), inplace = True)
...
153  all_data = pd.concat((X_train, test_data))
169  all_data = pd.get_dummies(all_data)
181  X_train = all_data[:n_train]
...
196  X_train, X_test, y_train, y_test = train_test_split(X_train, y_train, test_size=0.2, random_state=42)
...
232  svc_clf.fit(X_train, y_train)
233  pred_svc = svc_clf.predict(X_test)
```

Relative to the Kaggle train/test files, the `Age` median at line 90 looks
like it uses training data only. Every model is evaluated on the `X_test`
created at line 196, however, and that median was computed over those rows.
Anchoring the repair to the file-level split leaves the leak in place.
Anchoring it to line 196 removes it.
