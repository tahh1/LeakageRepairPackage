FROM python:3.9-slim

WORKDIR /kaggle

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

COPY ["Original dockerfiles/panamby_loan-default-prediction/requirements.txt", "."]
# pandas pinned <2 (1.5.3) because the notebook calls data.mean() on a
# dataframe that still has object-dtype (string) columns at that point
# (columns are dropped later) -- pandas >=2.0 raises TypeError on that,
# pandas 1.x silently excludes non-numeric columns. numpy pinned to match
# that pandas build's ABI.
# xgboost pinned <1.6: the notebook fits XGBClassifier directly on the raw
# integer `loss` column (~90 highly imbalanced classes, some with a single
# occurrence in the whole dataset) rather than binning to binary
# default/no-default. xgboost>=1.6 added a strict sklearn-wrapper check
# that the classes seen in y must be a dense contiguous 0..max range, which
# this real-world label distribution never satisfies (several intermediate
# loss values never occur at all), raising "Invalid classes inferred from
# unique values of y". Older xgboost's sklearn wrapper just LabelEncodes
# whatever classes are present and fits fine -- GaussianNB and
# RandomForestClassifier (no such check) already handle this same y
# without complaint, confirming this is a version-specific validation, not
# a real blocker.
# scikit-learn pinned <1.6 to pair with that old xgboost: sklearn 1.6
# replaced the old _get_tags()/_more_tags() estimator-tags protocol with
# __sklearn_tags__, and xgboost 1.5.2's XGBClassifier (built against the
# pre-1.6 protocol) doesn't implement it, so sklearn's own
# ClassifierMixin.__sklearn_tags__ -> super().__sklearn_tags__() chain
# breaks with "AttributeError: 'super' object has no attribute
# '__sklearn_tags__'" as soon as cross_val_score/is_classifier() touches
# the estimator. scikit-learn <1.6 still uses the old protocol, so it's
# compatible with this xgboost version.
RUN pip install --no-cache-dir -r requirements.txt

RUN mkdir -p /kaggle/working /kaggle/input
COPY ["Original notebooks/panamby_loan-default-prediction.ipynb", "/kaggle/working/"]
COPY ["Original dockerfiles/panamby_loan-default-prediction/train_v2.csv", "/kaggle/input/"]

EXPOSE 8888

CMD ["jupyter", "lab", "--ip=0.0.0.0", "--port=8888", "--no-browser", "--allow-root", \
     "--ServerApp.token=", "--ServerApp.password="]
