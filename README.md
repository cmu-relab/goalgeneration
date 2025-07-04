# Generative Goal Modeling - IEEE RE 2025
The Goal Generation study investigates the generation of simple goal models from interview transcripts.

# Authors
Ateeq Sharfuddin and Travis Breaux


# Summary
The material tied to the study are available under _study1_ and includes both datasets and notebooks.

The notebooks for the study use:
* Python 3.11.11
* SciKit Learn 1.6.1
* OpenAI model gpt-4o-2024-08-06

The file requirements.txt under _study1_ may be used to install the needed dependencies.

Note that the notebooks assume that the environment variable `OPENAI_API_KEY` has been configured.


# Citation
If you find our repository or paper useful, please cite us as:
```
@inproceedings{re2025generativegoalmodeling,
  title={Generative Goal Modeling},
  author={Sharfuddin, Ateeq and Breaux, Travis},
  booktitle={2025 IEEE 33rd International Requirements Engineering Conference (RE)},
  year={2025},
  organization={IEEE}
}
```

# Artifact Location

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.15664316.svg)](https://doi.org/10.5281/zenodo.15664316)

GitHub:
https://github.com/cmu-relab/goalgeneration/


# Details

We summarize each step, inputs and outputs, and motivations.

There are 34 transcripts, and these are available in the file transcripts.json under study1/data1. The transcripts.json
file has the following shape where `actions`, `goalmodel`, and `transcript` are lists. The `actions` field is a string
containing goals and actions the interviewer identified when interviewing the stakeholder, the `goalmodel` field
contains the goal graph the interviewer generated from the interview using https://github.com/cmu-relab/goalmodeling/, and the
`transcript` field contains the actual transcript of the interview. The `transcript` field is a list of lists,
where each inner list contains the turns of a single transcript. Each turn is a dictionary with the following keys:
- `turn`: The turn number in the transcript.
- `speaker`: The speaker of the turn, either 'interviewer' or 'interviewee'.
- `text`: The text of the turn.
- `goals`: goals mentioned in the turn.
- `actions`: actions mentioned in the turn.

The shape of transcripts and the shape of the 0th turn in transcript indexed at 0 is shown in the Python code below.

```python
>>> import json
>>> transcripts = json.load(open("transcripts.json"))
>>> transcripts.keys()
dict_keys(['transcript', 'actions', 'goalmodel'])
>>> transcripts['transcript'][0][0].keys()
dict_keys(['turn', 'speaker', 'text', 'goals', 'actions'])
```


The study consists of the following steps. The associated notebook(s) are specified for each step.

1. **Extract Goals from Transcripts.** This step may be performed in two modes: (a) whole transcript, or (b) moving
window. In whole transcript, goals are extracted from the entire transcript. This approach is prompted 10 times to
generate a diverse sample. In moving window, goals are extracted from a window of four speaker turns that moves at a
rate of two turns per prompt. Each window is prompted once.  These two modes correspond to the notebooks
`1. Extract Goals from Transcripts - Whole Transcript.ipynb` and
`1. Extract Goals from Transcripts - Moving Window.ipynb`, respectively. These notebooks use the `transcripts.json` as
input, and produce `extracted-moving.json` and `extracted-whole.json` as output, respectively.

2. **Analyze Extracted Goals.** This step computes basic statistics from extracted goals, including which speaker source
text was traced to each goal. This step corresponds to the notebook ` 2. Analyze Extracted Goals.ipynb`. The notebook
uses `extracted-moving.json` and `transcripts.json` as input, and produces `cache/n.y-generated-goals.csv` files,
where `n` is the transcript number, and `y` is the iteration. The notebook also produces `extracted-goals.json` and
`statistics.csv`.

3. **Cluster Goals from Repeated Sampling.** This step uses agglomerative clustering to cluster goals using Euclidean
distance. Goals are next sampled from each cluster for use in the model generation step. This step corresponds to the
notebook `3. Cluster Goals from Repeated Sampling.ipynb`. This notebook uses `extracted-goals.json` as input, and
produces `cache/clusters-n.json`, where n is the transcript number, `sampled-goals.json`, and `summarized.json`.

4. **Build Goal Refinement Graph.** This step prompts the model to generate a simple goal graph consisting of directed,
acyclic refinement edges. This step corresponds to the notebook `4. Build Goal Refinement Graph.ipynb`. This notebook
uses `summarized.json` as input, and produces `cache/refinements-n-implied.json`, where n is the transcript number, and
`models.json`.

5. **Extract Goal Graph from Python Code.** This step extracts the goal graph from the generated Python code. This step
corresponds to the notebook `5. Extract Goal Graph from Python Code.ipynb`. This notebook uses `models.json` as input,
and produces `graphs.json`.

The extracted goal graphs are represented using node and edge sets, which are then analyzed using ground truth
annotations of a stratified sample of labeled edges. The ground truth data consists of:

* **comps_unlabeled.csv** The unlabeled goal refinement relationship sample, which includes refinements coded with the
following keys: 'POSITIVE' means the refinement relationship was generated by the model in one direction; 'SYMMETRIC'
means the model generated the refinement relationship in both directions; and 'NEGATIVE' means the model did not
generate the refinement relationship. The file contains 250 positive, 50 symmetric and 250 negative relationships in
total.

* **comps_round1** The first round of 50 labeled relationships, randomly ordered.

* **comps_round2** The second round of an additional 50 labeled relationships, randomly ordered.

* **comps_final** The final labeled dataset.

The notebooks `6. Analyze Symmetries using Human Labels.ipynb` and `7. Analysis Symmetry Study Results.ipynb` are used
to analyze relationship symmetrics. The first notebook uses `comps_final.csv` as input and produces `comps1.json`,
`comps2.json`, and `comps3.json`. The second notebook uses these json and csv files to produce
`error_analysis.csv`.


The notebook tied to Research Question 1 is ` RQ1 - Trace Goals to Transcripts - Human Extracted.ipynb`.
An Excel spreadsheet `RQ2 - Mapping Human-authored goals to LLM-extracted goals.xlsx` is provided for Research Question 2.
Research Question 3 is evaluated in two notebooks `RQ3 - Evaluate Refinement Relations.ipynb` and
`RQ3 - Random Sample Refinement Relations.ipynb`.
