FROM python:3.11-slim

WORKDIR /home/jovyan/work

# graphviz (apt) provides the `dot` binary needed by both
# sklearn.tree.export_graphviz + pydotplus (cell using Image(graph.create_png()))
# and graphviz.Source(...).pipe(format='svg') — the pip packages alone only
# give the Python wrappers. build-essential covers any source builds.
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    graphviz \
    && rm -rf /var/lib/apt/lists/*

COPY ["Original dockerfiles/2021-09-12-nb_677/requirements.txt", "."]
RUN pip install --no-cache-dir -r requirements.txt

COPY ["Original notebooks/2021-09-12-nb_677.ipynb", "."]
COPY ["Original dockerfiles/2021-09-12-nb_677/weatherAUS.csv", "."]

# weatherAUS.csv is the Kaggle "Rain in Australia" dataset
# (jsphyg/weather-dataset-rattle-package, single file weatherAUS.csv,
# 145460 rows x 23 columns), fetched via the Kaggle MCP download_dataset tool.

EXPOSE 8888

CMD ["jupyter", "lab", "--ip=0.0.0.0", "--port=8888", "--no-browser", "--allow-root", \
     "--ServerApp.token=", "--ServerApp.password="]
