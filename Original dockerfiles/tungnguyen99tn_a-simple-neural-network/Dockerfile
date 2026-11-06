FROM python:3.11-slim

WORKDIR /kaggle

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

COPY ["Original dockerfiles/tungnguyen99tn_a-simple-neural-network/requirements.txt", "."]
# Notebook imports plain `keras` (Sequential/Dense/Adam/backend), predating
# TF2's tf.keras integration. tensorflow==2.15.0 bundles a matching
# keras==2.15.0 that still ships as the standalone `keras` package (pre
# keras-3 default in TF 2.16+), so `import keras...` resolves as the
# notebook expects. numpy<2 pinned since TF 2.15 requires numpy<2 anyway.
# pandas<2 pinned: the notebook does `iris_df.values[:, 1:5]` on a frame
# that includes the get_dummies() one-hot columns. pandas>=2.0 changed
# get_dummies()'s default dtype from uint8 to bool, which makes the whole
# frame's `.values` come back as an object array instead of float64 and
# breaks the Keras `model.fit()` call downstream.
RUN pip install --no-cache-dir -r requirements.txt

RUN mkdir -p /kaggle/working /kaggle/input/iris
COPY ["Original notebooks/tungnguyen99tn_a-simple-neural-network.ipynb", "/kaggle/working/"]
COPY ["Original dockerfiles/tungnguyen99tn_a-simple-neural-network/Iris.csv", "/kaggle/input/iris/"]

EXPOSE 8888

CMD ["jupyter", "lab", "--ip=0.0.0.0", "--port=8888", "--no-browser", "--allow-root", \
     "--ServerApp.token=", "--ServerApp.password="]
