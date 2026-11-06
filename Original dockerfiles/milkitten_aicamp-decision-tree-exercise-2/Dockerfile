FROM python:3.9-slim

WORKDIR /kaggle

# graphviz (apt) provides the `dot` binary needed by sklearn's
# tree.export_graphviz/graphviz.Source rendering; the pip package alone
# only gives the Python wrapper. build-essential covers any source builds.
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    graphviz \
    && rm -rf /var/lib/apt/lists/*

COPY ["Original dockerfiles/milkitten_aicamp-decision-tree-exercise-2/requirements.txt", "."]
# Pinned to numpy<2 / matplotlib 3.5.3 / matplotlib_inline 0.1.6 /
# seaborn 0.11.2: the notebook calls sns.countplot(x, hue=y) with the
# first argument positional, which seaborn 0.12+ repurposed for `data`
# (breaking change). Old matplotlib requires numpy<2 to avoid the
# NumPy 2.0 ABI break, and matplotlib_inline must be pinned to stay
# compatible with the old matplotlib's private RcParams API.
RUN pip install --no-cache-dir -r requirements.txt

RUN mkdir -p /kaggle/working /kaggle/input
COPY ["Original notebooks/milkitten_aicamp-decision-tree-exercise-2.ipynb", "/kaggle/working/"]
COPY ["Original dockerfiles/milkitten_aicamp-decision-tree-exercise-2/titanic.csv", "/kaggle/input/"]

# The notebook reads titanic.csv from ../input relative to the notebook,
# i.e. /kaggle/input/titanic.csv. It is baked into the image above; to
# override at runtime instead, bind-mount over it:
#   docker run -p 8888:8888 \
#     -v /path/to/titanic.csv:/kaggle/input/titanic.csv \
#     milkitten-aicamp-decision-tree-exercise-2

EXPOSE 8888

CMD ["jupyter", "lab", "--ip=0.0.0.0", "--port=8888", "--no-browser", "--allow-root", \
     "--ServerApp.token=", "--ServerApp.password="]
