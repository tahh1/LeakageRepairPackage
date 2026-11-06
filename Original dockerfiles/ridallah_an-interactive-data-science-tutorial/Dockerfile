FROM --platform=linux/amd64 python:3.6-slim

WORKDIR /kaggle

# python:3.6-slim's Debian repos are EOL and 404 on the live security
# mirror; pin apt to the debian snapshot archive so the base image stays
# buildable.
RUN sed -i \
    -e 's|http://deb.debian.org/debian|http://snapshot.debian.org/archive/debian/20211220T000000Z|' \
    -e 's|http://security.debian.org/debian-security|http://snapshot.debian.org/archive/debian-security/20211220T000000Z|' \
    /etc/apt/sources.list \
    && apt-get -o Acquire::Check-Valid-Until=false update \
    && apt-get -o Acquire::Check-Valid-Until=false install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

COPY ["Original dockerfiles/ridallah_an-interactive-data-science-tutorial/requirements.txt", "."]
# Pinned to Python 3.6 + numpy 1.19.5 + pandas 0.25.3 + scikit-learn 0.19.2 +
# matplotlib 3.0.3 because the notebook uses:
#   - `from sklearn.cross_validation import train_test_split, StratifiedKFold`
#     (module removed in scikit-learn 0.20; last available in the 0.19.x
#     line), including the old `StratifiedKFold(y, n_folds)` positional
#     signature.
#   - `from sklearn.preprocessing import Imputer` (removed in scikit-learn
#     0.22; present, unused at call-time, in 0.19.x).
#   - `full.append(test, ignore_index=True)` (DataFrame.append removed in
#     pandas 2.0; needs an old pandas).
#   - `titanic.corr()` on a still-mixed-dtype dataframe (pandas >=2.0 raises
#     without numeric_only=True; old pandas silently drops non-numeric
#     columns).
# scikit-learn 0.19.2 / pandas 0.25.3 only ship wheels up to Python 3.6/3.7,
# hence the old Python base image. Built for linux/amd64 explicitly because
# none of these old numpy/scipy/pandas/scikit-learn releases publish
# manylinux wheels for arm64/aarch64 -- on an Apple Silicon host, building
# without --platform=linux/amd64 makes pip fall back to source builds that
# fail. Requires Docker's amd64 emulation (QEMU/Rosetta).
RUN pip install --no-cache-dir -r requirements.txt

# Emulation gotcha: matplotlib.pyplot / pandas do fork()-based work (e.g.
# building a font cache) on first import. Doing that for the first time
# inside an already-multithreaded ipykernel process can deadlock forever
# under Docker Desktop's Rosetta x86_64-on-arm64 emulation. Warm the
# imports here, at build time, in a plain single-threaded script, so the
# caches already exist before any notebook imports them inside a kernel.
RUN python -c "\
import pandas, numpy, scipy, sklearn; \
from sklearn import tree, linear_model, neighbors, naive_bayes, svm, ensemble, preprocessing, feature_selection; \
from sklearn.cross_validation import train_test_split, StratifiedKFold; \
import matplotlib; \
import matplotlib.pyplot; \
import seaborn; \
print('warmed imports OK')"

RUN mkdir -p /kaggle/working /kaggle/input
COPY ["Original notebooks/ridallah_an-interactive-data-science-tutorial.ipynb", "/kaggle/working/"]
COPY ["Original dockerfiles/ridallah_an-interactive-data-science-tutorial/train.csv", "/kaggle/input/"]
COPY ["Original dockerfiles/ridallah_an-interactive-data-science-tutorial/test.csv", "/kaggle/input/"]

EXPOSE 8888

CMD ["jupyter", "lab", "--ip=0.0.0.0", "--port=8888", "--no-browser", "--allow-root", \
     "--ServerApp.token=", "--ServerApp.password="]
