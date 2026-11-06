FROM python:3.11-slim

WORKDIR /home/jovyan/work

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

COPY ["Original dockerfiles/2021-09-23-nb_2646/requirements.txt", "."]
RUN pip install --no-cache-dir -r requirements.txt

COPY ["Original notebooks/2021-09-23-nb_2646.ipynb", "."]
COPY ["Original dockerfiles/2021-09-23-nb_2646/hypothyroid.data", "."]

# The notebook reads "hypothyroid.data" from its own directory. Supply it at
# runtime with a bind mount, e.g.:
#   docker run -p 8888:8888 \
#     -v /path/to/hypothyroid.data:/home/jovyan/work/hypothyroid.data \
#     nb-2646

EXPOSE 8888

CMD ["jupyter", "lab", "--ip=0.0.0.0", "--port=8888", "--no-browser", "--allow-root", \
     "--ServerApp.token=", "--ServerApp.password="]
