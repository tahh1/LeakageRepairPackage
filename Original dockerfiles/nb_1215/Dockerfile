FROM python:3.11-slim

WORKDIR /home/jovyan/work

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

COPY ["Original dockerfiles/nb_1215/requirements.txt", "."]
RUN pip install --no-cache-dir -r requirements.txt

# Pre-download the NLTK stopwords corpus at build time so the image is
# fully self-contained and doesn't depend on runtime internet access. The
# notebook itself also calls nltk.download('stopwords') at runtime; this
# just makes that call a fast no-op instead of a hard requirement.
RUN python -m nltk.downloader stopwords

COPY ["Original notebooks/nb_1215.ipynb", "."]
COPY ["Original dockerfiles/nb_1215/spamham.csv", "."]

EXPOSE 8888

CMD ["jupyter", "lab", "--ip=0.0.0.0", "--port=8888", "--no-browser", "--allow-root", \
     "--ServerApp.token=", "--ServerApp.password="]
