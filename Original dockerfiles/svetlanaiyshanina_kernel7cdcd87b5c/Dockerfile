FROM python:3.11-slim

WORKDIR /home/jovyan/work

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

COPY ["Original dockerfiles/svetlanaiyshanina_kernel7cdcd87b5c/requirements.txt", "."]
RUN pip install --no-cache-dir -r requirements.txt

COPY ["Original notebooks/svetlanaiyshanina_kernel7cdcd87b5c.ipynb", "."]

# No dataset file needed: the notebook loads the Wine dataset via
# sklearn.datasets.load_wine(), which is bundled with scikit-learn itself.
# The one os.walk('/kaggle/input') boilerplate cell is informational only
# (prints nothing when the directory is absent/empty) and isn't relied on.

EXPOSE 8888

CMD ["jupyter", "lab", "--ip=0.0.0.0", "--port=8888", "--no-browser", "--allow-root", \
     "--ServerApp.token=", "--ServerApp.password="]
