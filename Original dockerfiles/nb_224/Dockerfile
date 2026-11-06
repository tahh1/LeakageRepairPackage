FROM python:3.11-slim

WORKDIR /home/jovyan/work

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

COPY ["Original dockerfiles/nb_224/requirements.txt", "."]
RUN pip install --no-cache-dir -r requirements.txt

COPY ["Original notebooks/nb_224.ipynb", "."]
COPY ["Original dockerfiles/nb_224/bank.csv", "."]

# bank.csv is the UCI Bank Marketing dataset (smaller variant, 4521 rows,
# 17 columns incl. target "y"), fetched from
# https://archive.ics.uci.edu/static/public/222/bank+marketing.zip
# (inner bank.zip -> bank.csv)

EXPOSE 8888

CMD ["jupyter", "lab", "--ip=0.0.0.0", "--port=8888", "--no-browser", "--allow-root", \
     "--ServerApp.token=", "--ServerApp.password="]
