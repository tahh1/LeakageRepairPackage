FROM --platform=linux/amd64 python:3.6-slim

WORKDIR /kaggle

# python:3.6-slim's Debian bullseye repos are EOL and 404 on the live
# security mirror; pin apt to the debian snapshot archive (already hinted
# at, commented out, in the image's own sources.list) so the base image
# stays buildable.
RUN sed -i \
    -e 's|http://deb.debian.org/debian|http://snapshot.debian.org/archive/debian/20211220T000000Z|' \
    -e 's|http://security.debian.org/debian-security|http://snapshot.debian.org/archive/debian-security/20211220T000000Z|' \
    /etc/apt/sources.list \
    && apt-get -o Acquire::Check-Valid-Until=false update \
    && apt-get -o Acquire::Check-Valid-Until=false install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

COPY ["Original dockerfiles/jonnyevans_multiclass-svm/requirements.txt", "."]
# Pinned to Python 3.6 + numpy 1.19.5 + pandas 0.25.3 + scikit-learn 0.19.2 +
# scikit-image 0.14.2 + matplotlib 3.0.3 because the notebook uses two
# APIs removed from modern releases:
#   - `from sklearn.cross_validation import train_test_split` (module
#     removed in scikit-learn 0.20; last available in the 0.19.x line)
#   - `train.ix[:,0]` (the `.ix` indexer, removed in pandas 1.0; still
#     present, just deprecated, in the pandas 0.2x line)
# scikit-learn 0.19.2 / pandas 0.25.3 only ship wheels up to Python 3.6/3.7,
# hence the old Python base image. Also note: this image is built for
# linux/amd64 explicitly because none of these old numpy/scipy/pandas/
# scikit-learn releases publish manylinux wheels for arm64/aarch64 --
# on an Apple Silicon host, building without --platform=linux/amd64 makes
# pip fall back to source builds that fail ("Broken toolchain: cannot link
# a simple C program"). Requires Docker's amd64 emulation (QEMU/Rosetta).
RUN pip install --no-cache-dir -r requirements.txt

# Emulation gotcha: on first use, matplotlib.pyplot builds its font cache
# (and pandas/scikit-image touch similar first-run codepaths) via a
# fork()-based worker under the hood. Doing that for the first time inside
# an ipykernel process (which already has a background zmq heartbeat
# thread) deadlocks forever under Docker Desktop's Rosetta x86_64-on-arm64
# emulation -- even though the exact same imports return instantly in a
# plain, non-threaded `python -c` script. Fix: warm every such import here,
# at build time, in a plain script, so the caches (e.g.
# ~/.cache/matplotlib/fontlist-v300.json) already exist before any
# notebook ever imports them inside a kernel.
RUN python -c "\
import pandas, numpy, scipy, sklearn; \
from sklearn import svm, decomposition, preprocessing; \
import matplotlib; \
import matplotlib.pyplot; \
import skimage; \
from skimage.transform import rotate; \
print('warmed imports OK')"

RUN mkdir -p /kaggle/working /kaggle/input
COPY ["Original notebooks/jonnyevans_multiclass-svm.ipynb", "/kaggle/working/"]
COPY ["Original dockerfiles/jonnyevans_multiclass-svm/train.csv", "/kaggle/input/"]
COPY ["Original dockerfiles/jonnyevans_multiclass-svm/test.csv", "/kaggle/input/"]

EXPOSE 8888

CMD ["jupyter", "lab", "--ip=0.0.0.0", "--port=8888", "--no-browser", "--allow-root", \
     "--ServerApp.token=", "--ServerApp.password="]
