FROM python:3.11-slim

WORKDIR /home/jovyan/work

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

COPY ["Original dockerfiles/2021-09-09-nb_1731/requirements.txt", "."]
RUN pip install --no-cache-dir -r requirements.txt

COPY ["Original notebooks/2021-09-09-nb_1731.ipynb", "."]
COPY ["Original dockerfiles/2021-09-09-nb_1731/Pickles", "./Pickles"]

# The notebook's last cell pickles the trained model + a results dataframe to
# Models/best_mnbc.pickle and Models/df_models_mnbc.pickle (relative path) -
# create the directory so that write succeeds.
RUN mkdir -p Models

EXPOSE 8888

CMD ["jupyter", "lab", "--ip=0.0.0.0", "--port=8888", "--no-browser", "--allow-root", \
     "--ServerApp.token=", "--ServerApp.password="]
