# Technical Brief: Automating the Daily Sensor Update

**To:** Operations Manager
**From:** Data/Analytics Team
**Re:** Replacing the manual daily sensor upload with an automated pipeline

## The problem

Every day, someone manually opens the sensor export, checks it for obvious
errors, and copies the numbers into our tracking database. This takes time,
it's easy to make a copy-paste mistake, and there's no record of exactly
what was checked or when.

## What we built

We built a small program (`run_pipeline.py`) that does this job automatically,
every day, without anyone touching it. It has four steps:

1. **Extract** — Pulls the day's sensor readings from the export file (or an
   API, if we switch data sources later).
2. **Transform** — Cleans the data: removes duplicate readings, standardizes
   sensor names and status codes, and drops rows with impossible values
   (e.g., negative pressure).
3. **Quality gate** — Before anything touches the database, the pipeline
   checks the data against six rules (no missing timestamps or sensor IDs,
   no duplicate timestamps, pressure between 0–200 psi, temperature between
   -20–120°C, and only recognized status codes). If any rule fails, **the
   pipeline stops** and nothing bad gets loaded — no manual review needed to
   catch the error.
4. **Load** — Safely writes the day's clean data into our database. If the
   pipeline is run twice for the same day (e.g., after a fix), it replaces
   that day's rows instead of creating duplicates.

## Why this matters

- **Fewer errors:** Bad data is caught automatically instead of slipping
  through to reports.
- **Time saved:** No more manual copy-paste each morning; the job runs on a
  schedule (6:00 AM daily via cron/Task Scheduler — see
  `docs/automation_proof.md`).
- **Auditable:** Every run is logged to `pipeline.log` with start time, row
  counts, and any errors, so we can always answer "what happened on a given
  day?"
- **Safe to re-run:** If we ever need to reprocess a day (e.g., a
  late-arriving correction), running the pipeline again does not create
  duplicate records — it simply replaces that day's snapshot.

## What we tested

- A normal day's file runs end-to-end and lands in the database.
- Running the same day twice in a row leaves the same row count — no
  duplicates.
- A deliberately broken file (bad pressure readings, missing sensor ID,
  duplicate timestamps) is correctly **rejected** before it reaches the
  database, with a clear message in the log: `Quality gate failed. Load step
  halted.`

## Next steps

- Point `SOURCE_TYPE`/`API_URL` at the live sensor API once credentials are
  available (currently reads from a CSV export).
- Add an alert (email/Slack) when the quality gate halts a run, so someone is
  notified same-day instead of discovering it later.
