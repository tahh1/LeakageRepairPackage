For every notebook in Experiments/E1/Experimental setup/Fixed notebooks tracker.csv where assigned_to == "me" and status == "pending", spawn one subagent per notebook, in parallel where possible.

Give each subagent exactly this prompt, substituting the notebook and Dockerfile paths, and nothing else — no extra context, hints, or instructions about what to look for:

"You are reviewing the Jupyter notebook at {notebook_path}.

Find and fix any machine learning data leakage in this notebook.

The notebook's environment is defined in the Dockerfile at {dockerfile_path}. You may build and run this Docker environment to execute the notebook as you work, if that helps you find or verify the fix.

Make your edits directly in the notebook file. When you are done, summarize in a few sentences what you found and what you changed. If you found nothing, say so explicitly."

After each subagent finishes: save its full transcript to Experiments/E1/transcripts/{notebook_id}.log, update that row's subagent_summary in the tracker with its summary, and set status to completed.