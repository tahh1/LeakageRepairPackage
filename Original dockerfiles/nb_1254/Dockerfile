FROM python:3.11-slim

WORKDIR /home/jovyan/work

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

COPY ["Original dockerfiles/nb_1254/requirements.txt", "."]
# Notebook does `from keras.models import Sequential` / `from keras.layers
# import Dense`, the bare pre-tf.keras import style. tensorflow==2.15.0 is
# the last TF release that also installs a standalone keras==2.15.0 package
# matching that old top-level import API (TF 2.16+ makes bare `keras`
# resolve to Keras 3, a different surface). TF 2.15 requires numpy<2.
RUN pip install --no-cache-dir -r requirements.txt

RUN mkdir -p /home/jovyan/work/input
COPY ["Original notebooks/nb_1254.ipynb", "/home/jovyan/work/"]
COPY ["Original dockerfiles/nb_1254/input/Churn_Modelling.csv", "/home/jovyan/work/input/"]

EXPOSE 8888

CMD ["jupyter", "lab", "--ip=0.0.0.0", "--port=8888", "--no-browser", "--allow-root", \
     "--ServerApp.token=", "--ServerApp.password="]
