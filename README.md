# Nimbus Streaming — Standard tier pricing decision

Recommendation, analysis, and supporting documentation for the proposed $12.99 → $14.99
Standard tier price increase.

## Contents

| File | What it is |
|---|---|
| `nimbus_decision_starter.ipynb` | Full analysis: IPW/AIPW pilot correction, Bayesian updating, cost-benefit model, Monte Carlo, decision tree, decision theory, sensitivity. All cells executed with outputs visible. |
| `decision_memo.md` | One-page memo to the CFO. |
| `assumptions_and_sources.md` | Inputs, data lineage, methodological choices, and known limitations. |
| `requirements.txt` | Python dependencies. |

## Expected folder layout

The notebook reads the five CSVs from `../data/`. Keep this structure:

```
project/
├── data/
│   ├── pilot_data.csv
│   ├── industry_pricing_history.csv
│   ├── pre_announcement_survey.csv
│   ├── finance_forecast.csv
│   └── wtp_segments.csv
└── starter/
    ├── nimbus_decision_starter.ipynb
    ├── decision_memo.md
    ├── assumptions_and_sources.md
    └── requirements.txt
```

If your layout differs, the notebook falls back to searching `./data`, `.`, and `../../data`
before raising a clear error, so it will also run with the CSVs sitting next to it.

## Set up the environment

Python 3.10 or newer. From the project root:

**macOS / Linux**

```bash
python -m venv .venv
source .venv/Scripts/activate
python -m pip install --upgrade pip
python -m pip install -r starter/requirements.txt
```

**Windows (PowerShell)**

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r starter\requirements.txt
```

If PowerShell blocks the activation script, run
`Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass` first, or use
`.\.venv\Scripts\activate.bat` from `cmd.exe`.

Register the environment as a Jupyter kernel so the notebook picks it up:

```bash
python -m ipykernel install --user --name nimbus --display-name "Python (nimbus)"
```

Leave the environment with `deactivate`. To start over, delete the `.venv` folder.

<details>
<summary>Conda alternative</summary>

```bash
conda create -n nimbus python=3.12
conda activate nimbus
python -m pip install -r starter/requirements.txt
python -m ipykernel install --user --name nimbus --display-name "Python (nimbus)"
```
</details>

## Run the analysis

Interactively:

```bash
jupyter lab starter/nimbus_decision_starter.ipynb
```

Select the **Python (nimbus)** kernel, then *Run → Restart Kernel and Run All Cells*.
Runtime is roughly 1–2 minutes; the two bootstrap loops (500 IPW resamples, 300 AIPW
resamples, each refitting the propensity model) account for most of it.

Headless, to confirm it runs top-to-bottom from a fresh kernel:

```bash
cd starter
jupyter execute nimbus_decision_starter.ipynb --inplace
```

## Reproducibility

Every random draw comes from a single seeded generator, `np.random.default_rng(7)`, consumed
in a fixed order: the IPW bootstrap, then the AIPW bootstrap, then the 10,000-draw Monte Carlo.
Re-running from a fresh kernel reproduces every number in the memo exactly. Headline values to
check: IPW lift **3.70 pp**, posterior **3.81 ± 0.25 pp**, `Full` expected profit **$15.1M**,
break-even lift **5.11 pp**, `RECOMMENDED = "Full"`.
