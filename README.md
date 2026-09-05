# Industrializing the Daily Operations Pipeline

This lab turns a manual daily sensor update into a modular, repeatable Python pipeline.

## Files

- `run_pipeline.py` - modular extract, transform, quality gate, and load script.
- `data/sensor_data.csv` - clean sample input (12 rows, one snapshot day).
- `data/sensor_data_bad_example.csv` - intentionally bad input for testing the quality gate (negative pressure, out-of-range pressure, missing sensor ID, duplicate timestamp, unrecognized status).
- `.env` - local configuration for this lab (not committed; see `.gitignore`).
- `.env.example` - safe template for GitHub.
- `requirements.txt` - Python dependencies.
- `docs/automation_proof.md` - cron and Windows Task Scheduler proof instructions.
- `docs/technical_brief.md` - plain-English manager memo explaining the pipeline.
- `output/` - destination for the generated SQLite database (git-ignored; created on first run).

## Push this repo to GitHub

```bash
git init
git add .
git commit -m "Initial commit: modular ETL pipeline with quality gate"
git branch -M main
git remote add origin https://github.com/<your-username>/week4_etl_pipeline.git
git push -u origin main
```

## Setup

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
copy .env.example .env
# edit .env if you want to change paths, then:
python run_pipeline.py
```

macOS/Linux equivalent:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
python run_pipeline.py
```

> Note: the quality gate uses Great Expectations if it's installed
> (`pip install -r requirements.txt` includes it). If Great Expectations
> isn't available in your environment, `run_pipeline.py` automatically falls
> back to five equivalent local checks so the halt behavior still works.

## Quality Gate Test

To prove the pipeline halts on bad data, temporarily set this in `.env`:

```text
SOURCE_CSV=data/sensor_data_bad_example.csv
```

Then rerun:

```powershell
python run_pipeline.py
```

The run should stop before loading to the database.

## Idempotency Test

Run the pipeline twice with the clean file:

```powershell
python run_pipeline.py
python run_pipeline.py
```

The script deletes the existing rows for the configured `SNAPSHOT_DATE` and reloads that day. Re-running does not create duplicate daily records.

## Logging

Every run appends to `pipeline.log` (git-ignored) with a timestamp, start/end
markers, row counts at each stage, and a full traceback if the run fails.

## Automation

See `docs/automation_proof.md` for the cron / Windows Task Scheduler
configuration used to run this pipeline daily without manual intervention.

## Manager summary

See `docs/technical_brief.md` for a plain-English explanation of what this
pipeline does and why, written for a non-technical audience.

## Interactive walkthrough

Open `docs/pipeline_simulation.html` in any browser for an animated,
click-through walkthrough of the pipeline (no install required). It has two
buttons — "Run clean pipeline" and "Run bad-data test" — that step through
Extract → Transform → Quality Gate → Load → Log with a live log panel, so you
can show a reviewer or manager how the quality gate halts a bad run without
running any code.
