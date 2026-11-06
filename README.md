# Replication package: agent-based repair of data leakage in ML notebooks

This package contains the material and results for our three leakage-repair experiments (E1, E2, E3). In each experiment, a coding agent (Claude Code) was asked to find and fix data leakage in the same 19 Jupyter notebooks.

| Experiment | What the agent received |
|---|---|
| **E1** | The notebook and its Dockerfile |
| **E2** | E1, plus the static-analysis detection report for the notebook |
| **E3** | E2, plus written domain guidance on leakage repair |

The exact prompts are in `Experiments/E<n>/Experimental setup/Claude fix prompt.md`.

## Contents

```
Original notebooks/              the 19 notebooks before any fix
Original dockerfiles/<id>/       Dockerfile, requirements.txt and data for each notebook
RECONSTRUCTION_CHANGE_AUDIT.md   changes needed to make the original notebooks run
RULE_DERIVATION_EVIDENCE.md      where the E3 repair rules come from
Experiments/
  detection_reports/<id>.json    detection reports given to the agent in E2 and E3
  E1/ E2/ E3/
    Fixed notebooks/<id>.ipynb   the notebook after the agent's fix
    Experimental setup/
      Claude fix prompt.md       the prompt given to the agent
      Fixed notebooks tracker.csv  the agent's own summary and how it validated its fix
    Experiment results.csv       our assessment of each fix
```

## Docker environments

`Original dockerfiles/` has one environment per notebook. All three experiments use these same environments without changes. Only the notebook differs.

Build from the package root:

```
docker build -f "Original dockerfiles/<id>/Dockerfile" -t <id> .
docker run -p 8888:8888 <id>
```

This opens JupyterLab on port 8888 with the original notebook. To run a fixed notebook in the same environment, mount it over the original. The target path is the destination of the notebook's `COPY` line in the Dockerfile: either `/kaggle/working/<id>.ipynb` or `/home/jovyan/work/<id>.ipynb`.

```
docker run -p 8888:8888 \
  -v "$PWD/Experiments/E2/Fixed notebooks/<id>.ipynb:/kaggle/working/<id>.ipynb" <id>
```

### Data files not included

Four data files are larger than GitHub's 100 MB limit and are not in this repository. Download them from Kaggle and place them at the paths below before building. Competition data requires accepting the competition rules on Kaggle.

| Notebook | Path in this package | Source |
|---|---|---|
| `artgor_basic-eda` | `Original dockerfiles/artgor_basic-eda/train.csv` | Kaggle competition `pubg-finish-placement-prediction`, file `train_V2.csv` (renamed) |
| `artgor_basic-eda` | `Original dockerfiles/artgor_basic-eda/test.csv` | same competition, file `test_V2.csv` (renamed) |
| `panamby_loan-default-prediction` | `Original dockerfiles/panamby_loan-default-prediction/train_v2.csv` | Kaggle competition `loan-default-prediction`, file `train_v2.csv` |
| `nb_174` | `Original dockerfiles/nb_174/creditcard.csv` | Kaggle dataset `mlg-ulb/creditcardfraud`, file `creditcard.csv` |

## Results

`Experiments/E<n>/Experiment results.csv` holds our assessment of each fix in that experiment. There is one row per model pair, meaning one training call together with one evaluation call that uses the trained model. The notebook ID and notebook-level columns appear only on a notebook's first row.

| Column | Level | Meaning |
|---|---|---|
| `notebook_id`, `#pairs`, `pairs` | notebook / pair | notebook, its number of model pairs, and the pair ID from the detector |
| `#sources` | pair | number of distinct leakage sources in the pair |
| `overlap GT (detection)` | pair | 1 if the pair has overlap leakage (train/evaluation data overlap) |
| `overlap GT (ToCorrect)` | pair | 1 if that overlap should be corrected. Evaluating on exactly the training data is treated as an intentional fit check, not leakage to correct |
| `preproc GT` | pair | 1 if the pair has preprocessing leakage |
| `Fixed overlap`, `Fixed preproc` | pair | 1 if all leakage of that kind in the pair was fixed |
| `Fix cplt %sources` | pair | correctable sources fixed / total correctable sources |
| `reason` | pair | the leakage mechanisms in plain words |
| `sources on report` | pair | number of sources the detection report pointed to |
| `miss cause`, `miss case explanation` | pair | why an incomplete fix fell short |
| `notebook executes?` | notebook | whether the fixed notebook runs |
| `semantics preserved`, `changed semantics` | notebook | whether the fix preserves what the notebook originally did, and the blocking changes if not |
| `drift_tags`, `notes on drift tags` | notebook | non-blocking side effects of the fix |
| `extra_fixes` | notebook | genuine leakage fixes beyond the annotated sources |
| `excluded_changes` | notebook | environment repairs, not scored |
| `revision_notes` | notebook | assessment notes |
| `Fully acceptable` | notebook | executes, fixes every correctable source, and preserves semantics |
| `subagent_summary`, `agent_validation` | notebook | the agent's own summary and how it validated its fix |

Three notebooks contain only overlap that should not be corrected: `nb_299`, `2021-09-09-nb_1731` and `bbose71_kings-county-housing-done-in-class`. They serve as negative cases that test whether the agent leaves such code alone.
