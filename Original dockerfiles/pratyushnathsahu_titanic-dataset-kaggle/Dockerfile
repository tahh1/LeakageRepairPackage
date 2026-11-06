FROM python:3.11-slim

WORKDIR /kaggle

# xgboost needs libgomp1 at runtime for OpenMP; build-essential covers any
# source builds triggered by pip.
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    libgomp1 \
    && rm -rf /var/lib/apt/lists/*

COPY ["Original dockerfiles/pratyushnathsahu_titanic-dataset-kaggle/requirements.txt", "."]
# pandas<2 (+ numpy<2 for ABI match) is required: the notebook does
# combined_df["Embarked"] = Embarked_imputer.fit_transform(combined_df[["Embarked"]])
# — assigning a SimpleImputer's (n,1) object-dtype ndarray (Embarked is a
# string column) back into a single DataFrame column. On pandas 3.x this
# hits a regression in DataFrame._sanitize_column / lib.maybe_convert_objects
# ("ValueError: Buffer has wrong number of dimensions (expected 1, got 2)").
# pandas 1.5.3 handles the same (n,1) object array assignment fine.
RUN pip install --no-cache-dir -r requirements.txt

RUN mkdir -p /kaggle/working /kaggle/input
COPY ["Original notebooks/pratyushnathsahu_titanic-dataset-kaggle.ipynb", "/kaggle/working/"]
COPY ["Original dockerfiles/pratyushnathsahu_titanic-dataset-kaggle/train.csv", "/kaggle/input/"]
COPY ["Original dockerfiles/pratyushnathsahu_titanic-dataset-kaggle/test.csv", "/kaggle/input/"]

# Standard Kaggle Titanic competition dataset (titanic), fetched via the
# Kaggle MCP download_competition_data_files tool. Notebook reads
# "../input/train.csv" / "../input/test.csv" relative to the notebook's
# working directory (/kaggle/working), i.e. /kaggle/input here.

EXPOSE 8888

CMD ["jupyter", "lab", "--ip=0.0.0.0", "--port=8888", "--no-browser", "--allow-root", \
     "--ServerApp.token=", "--ServerApp.password="]
