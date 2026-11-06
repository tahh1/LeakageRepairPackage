FROM python:3.9-slim

WORKDIR /kaggle

# lightgbm needs libgomp1 at runtime; build-essential covers any source builds.
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    libgomp1 \
    && rm -rf /var/lib/apt/lists/*

COPY ["Original dockerfiles/artgor_basic-eda/requirements.txt", "."]
# Pinned to plotly 3.x / matplotlib 3.5 / Python 3.9 because the notebook
# uses the pre-4.0 `plotly.plotly` API and the removed `seaborn-notebook`
# matplotlib style.
RUN pip install --no-cache-dir -r requirements.txt

RUN mkdir -p /kaggle/working /kaggle/input
COPY ["Original notebooks/artgor_basic-eda.ipynb", "/kaggle/working/"]
COPY ["Original dockerfiles/artgor_basic-eda/train.csv", "/kaggle/input/"]
COPY ["Original dockerfiles/artgor_basic-eda/test.csv", "/kaggle/input/"]
COPY ["Original dockerfiles/artgor_basic-eda/sample_submission.csv", "/kaggle/input/"]




# This is a Kaggle PUBG "Finish Placement" kernel. It expects
# train.csv, test.csv and sample_submission.csv one level up from the
# notebook, i.e. at /kaggle/input. Supply it at runtime with a bind mount:
#   docker run -p 8888:8888 -v /path/to/pubg-data:/kaggle/input artgor-basic-eda

EXPOSE 8888

CMD ["jupyter", "lab", "--ip=0.0.0.0", "--port=8888", "--no-browser", "--allow-root", \
     "--ServerApp.token=", "--ServerApp.password="]
