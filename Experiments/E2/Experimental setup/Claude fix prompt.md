For every notebook in Experiments/E2/Experimental setup/Fixed notebooks tracker.csv where assigned_to == "me" and status == "pending", spawn one subagent per notebook, in parallel where possible.

Give each subagent exactly this prompt, substituting the notebook, report and Dockerfile paths, and nothing else — no extra context, hints, or instructions about what to look for:

"You are reviewing the Jupyter notebook at {notebook_path}.

Find and fix any machine learning data leakage in this notebook.

A static analysis tool was previously run over this notebook. Its report is at {report_path}. Read it before you start.

The report is a JSON object whose single key, "Model pairs with data leakage", holds a list of model pairs. A model pair is one model-training call together with one evaluation call that uses the model it trained — a notebook that trains a model once and evaluates it twice yields two pairs. The tool analyzes each pair separately. Each entry has:

- "snippet": the program slice for that pair — the subset of the notebook's statements that affect the data flowing into its training and evaluation calls. It is not the whole notebook, and its statements are not necessarily adjacent in it. Each line is prefixed with its line number in the notebook, so the numbers are not contiguous.
- "model info": which call trains the model and which evaluates it, with their line numbers.
- "preproc_leakage_detected" / "overlap_leakage_detected": whether each kind was found for that pair.
- "preproc_leakage_lines" / "overlap_leakage_lines": the lines held responsible, each with an explanation.

Because the pairs are analyzed independently, one underlying problem affecting several pairs is reported separately under each of them, so the same line can appear under more than one pair and a single edit may resolve findings listed under several.

The notebook's environment is defined in the Dockerfile at {dockerfile_path}. You may build and run this Docker environment to execute the notebook as you work, if that helps you find or verify the fix.

Make your edits directly in the notebook file. When you are done, summarize in a few sentences what you found and what you changed. If you found nothing, say so explicitly."

After each subagent finishes: save its full transcript to Experiments/E2/transcripts/{notebook_id}.log, update that row's subagent_summary in the tracker with its summary, and set status to completed.