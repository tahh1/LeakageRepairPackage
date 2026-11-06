FROM python:3.11-slim

WORKDIR /home/jovyan/work

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

COPY ["Original dockerfiles/nb_225/requirements.txt", "."]
RUN pip install --no-cache-dir -r requirements.txt

COPY ["Original notebooks/nb_225.ipynb", "."]
COPY ["Original dockerfiles/nb_225/car.data", "."]

# car.data is the UCI Car Evaluation dataset (1728 rows, 6 unnamed feature
# columns + class label), fetched from
# https://archive.ics.uci.edu/ml/machine-learning-databases/car/car.data

EXPOSE 8888

CMD ["jupyter", "lab", "--ip=0.0.0.0", "--port=8888", "--no-browser", "--allow-root", \
     "--ServerApp.token=", "--ServerApp.password="]
