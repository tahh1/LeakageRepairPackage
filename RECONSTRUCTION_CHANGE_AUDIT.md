# Reconstruction Change Audit

This document lists the interventions that were needed to make the notebooks in this replication package run in their Docker environments. Four of the 19 notebooks needed an intervention. The other 15 ran without changes.

Intervention classes:

- **Code edit:** a notebook line or cell was added, changed, removed, or moved.
- **Data intervention:** a data file was replaced with a matching source, renamed, or reformatted without changing the notebook code.
- **Environment intervention:** compatible library versions or build-time resources were supplied without changing the notebook code.

## Summary

| Notebook | Intervention class | Reconstruction issue | Leakage and modeling assessment |
|---|---|---|---|
| `artgor_basic-eda` | Code + data | Revised competition files and invalid no-op `KFold` argument | Same unshuffled folds; leakage code unchanged |
| `pratyushnathsahu_titanic-dataset-kaggle` | Environment | Modern pandas rejected a legacy assignment | Code and all leakage sources retained |
| `brendalf_data-science-for-social-good-in-practice` | Data + environment | Legacy plotting behavior and unavailable original distribution | Matching dataset schema and row count; code unchanged |
| `nb_1215` | Data + environment | Deleted dataset upload and header capitalization mismatch | Data rows unchanged; notebook code unchanged |

No intervention added or removed a leakage source, and none changed a model setting.

## Details: notebooks with code edits

### `artgor_basic-eda`

**Notebook behavior.** The notebook performs exploratory analysis on the Kaggle PUBG data and trains a LightGBM regressor using ten-fold `KFold` cross-validation.

**Failures.** Kaggle's revised competition release uses `train_V2.csv` and `test_V2.csv` rather than the original filenames. In addition, the notebook constructed `KFold` with `shuffle=False` and `random_state=42`. The random state has no effect when folds are not shuffled; older scikit-learn versions ignored it with a warning, whereas modern versions raise an error.

**Interventions.** Supplied the revised V2 data files and removed the ineffective `random_state` argument, leaving `KFold(n_splits=n_fold, shuffle=False)`.

**Audit conclusion.** The folds remain unshuffled and identical in construction. The revised files preserve every column used by the notebook, although reported scores need not match the original release. The leakage sources are expressed in the notebook code and were not changed.

## Details: data- or environment-only interventions

### `pratyushnathsahu_titanic-dataset-kaggle`

**Issue.** Modern pandas rejected assignment of a one-column two-dimensional array returned by `SimpleImputer` into a single text column.

**Intervention.** Used pandas and NumPy versions compatible with the notebook's original assignment behavior. No notebook code was edited.

**Audit conclusion.** All combined-data filling, binning, and encoding operations remain exactly as authored, including their leakage sources.

### `brendalf_data-science-for-social-good-in-practice`

**Issues.** A plotting call relied on behavior rejected by matplotlib 3.3 and later. The original Udacity CharityML distribution was not available from its original source.

**Interventions.** Used the compatible legacy plotting stack and a Kaggle mirror with the same columns and 45,222-row count as the expected dataset.

**Audit conclusion.** The notebook code is unchanged, and the replacement data matches the expected schema and size. The intervention does not add or remove a leakage source.

### `nb_1215`

**Issue.** The Kaggle dataset referenced by the notebook was no longer available. A matching re-upload used capitalized `Text` and `Spam` headers, while the notebook expects lowercase names.

**Interventions.** Supplied the matching 5,728-row re-upload and lowercased only its header row. Downloaded the NLTK stopword list at image-build time.

**Audit conclusion.** No data row or notebook line was changed. The replacement has the same content and shape reported for the original, although byte-for-byte identity with the deleted upload cannot be established.
