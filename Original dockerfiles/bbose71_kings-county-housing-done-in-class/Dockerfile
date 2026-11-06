FROM python:3.11-slim

WORKDIR /kaggle

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

COPY ["Original dockerfiles/bbose71_kings-county-housing-done-in-class/requirements.txt", "."]
RUN pip install --no-cache-dir -r requirements.txt

RUN mkdir -p /kaggle/working /kaggle/input
COPY ["Original notebooks/bbose71_kings-county-housing-done-in-class.ipynb", "/kaggle/working/"]
COPY ["Original dockerfiles/bbose71_kings-county-housing-done-in-class/kc_house_data.csv", "/kaggle/input/"]

EXPOSE 8888

CMD ["jupyter", "lab", "--ip=0.0.0.0", "--port=8888", "--no-browser", "--allow-root", \
     "--ServerApp.token=", "--ServerApp.password="]
