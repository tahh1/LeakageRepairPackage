FROM --platform=linux/amd64 python:3.8-slim

WORKDIR /kaggle

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

COPY ["Original dockerfiles/brendalf_data-science-for-social-good-in-practice/requirements.txt", "."]
# Cell In[13] does `plt.errorbar(y=df_train.income.mean(), x=np.arange(-1, 97, 1), ...)`
# — a scalar y broadcast against an array x, used to draw a dashed reference
# line. Verified this is a real version-dependent matplotlib behavior change,
# not a notebook bug: matplotlib<=3.2.2 silently broadcasts/tolerates the
# mismatched lengths, matplotlib>=3.3.0 added a strict length check and raises
# `ValueError: 'x' and 'y' must have the same size`. Pin matplotlib==3.2.2,
# which in turn needs numpy<2 (old-matplotlib/NumPy-2.0 ABI break),
# matplotlib_inline==0.1.6 (old matplotlib lacks the private RcParams API
# newer matplotlib_inline calls), and forces this python:3.8-slim base (the
# newest Python matplotlib 3.2.2 still ships an amd64 wheel for). seaborn==
# 0.11.2 pairs with the old matplotlib and is the known-good pin for
# old-style seaborn calls elsewhere in this repo, but its FacetGrid.map
# internals call the removed `DataFrame.iteritems()` (dropped in pandas 2.0),
# raising `AttributeError: 'DataFrame' object has no attribute 'iteritems'`
# on the age KDE FacetGrid cell — pin pandas==1.5.3 to keep it available.
RUN pip install --no-cache-dir -r requirements.txt

# Apple Silicon / Rosetta fork() deadlock warm-up: first-importing pandas /
# matplotlib.pyplot from inside an already-multithreaded ipykernel process
# can hang forever under x86_64-on-arm64 emulation. Do the fork-triggering
# work once, single-threaded, at build time instead.
RUN python -c "import pandas; import matplotlib; matplotlib.use('Agg'); import matplotlib.pyplot; import seaborn; import sklearn; import xgboost"

RUN mkdir -p /kaggle/working /kaggle/input/udacity-mlcharity-competition
COPY ["Original notebooks/brendalf_data-science-for-social-good-in-practice.ipynb", "/kaggle/working/"]
COPY ["Original dockerfiles/brendalf_data-science-for-social-good-in-practice/census.csv", "/kaggle/input/udacity-mlcharity-competition/"]
COPY ["Original dockerfiles/brendalf_data-science-for-social-good-in-practice/test_census.csv", "/kaggle/input/udacity-mlcharity-competition/"]

# The notebook reads ../input/udacity-mlcharity-competition/{census,test_census}.csv
# relative to the notebook at /kaggle/working, i.e.
# /kaggle/input/udacity-mlcharity-competition/{census,test_census}.csv. Data is
# baked into the image above; to override at runtime instead, bind-mount over it:
#   docker run -p 8888:8888 \
#     -v /path/to/udacity-mlcharity-competition:/kaggle/input/udacity-mlcharity-competition \
#     brendalf-data-science-for-social-good-in-practice

EXPOSE 8888

CMD ["jupyter", "lab", "--ip=0.0.0.0", "--port=8888", "--no-browser", "--allow-root", \
     "--ServerApp.token=", "--ServerApp.password="]
