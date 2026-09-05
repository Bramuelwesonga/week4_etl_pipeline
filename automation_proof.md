# Automation Proof

This document shows how `run_pipeline.py` is scheduled to run automatically once
per day, and provides the exact configuration used (or to be used) for grading.

## Option A — Linux/macOS (cron)

1. Open the crontab editor:

   ```bash
   crontab -e
   ```

2. Add the following line to run the pipeline every day at 6:00 AM using the
   project's virtual environment:

   ```cron
   0 6 * * * cd /home/user/week4_etl_pipeline && /home/user/week4_etl_pipeline/.venv/bin/python run_pipeline.py >> cron_stdout.log 2>&1
   ```

   Field breakdown:

   | Field | Value | Meaning |
   |---|---|---|
   | Minute | `0` | On the hour |
   | Hour | `6` | 6 AM |
   | Day of month | `*` | Every day |
   | Month | `*` | Every month |
   | Day of week | `*` | Every weekday |

3. Confirm the job was registered:

   ```bash
   crontab -l
   ```

   Expected output includes:

   ```text
   0 6 * * * cd /home/user/week4_etl_pipeline && /home/user/week4_etl_pipeline/.venv/bin/python run_pipeline.py >> cron_stdout.log 2>&1
   ```

## Option B — Windows (Task Scheduler)

1. Open **Task Scheduler** → **Create Basic Task…**
2. **Name:** `Week4 ETL Pipeline`
3. **Trigger:** Daily, start time `6:00:00 AM`
4. **Action:** Start a program
   - **Program/script:**
     `C:\Users\<user>\week4_etl_pipeline\.venv\Scripts\python.exe`
   - **Add arguments:** `run_pipeline.py`
   - **Start in:** `C:\Users\<user>\week4_etl_pipeline`
5. Finish the wizard, then open the task's **Properties** and check
   **Run whether user is logged on or not** so it runs unattended.
6. Equivalent command-line registration (`schtasks`), which produces the same
   scheduled task and can be pasted into a terminal as proof:

   ```powershell
   schtasks /Create /SC DAILY /ST 06:00 /TN "Week4 ETL Pipeline" ^
     /TR "C:\Users\<user>\week4_etl_pipeline\.venv\Scripts\python.exe C:\Users\<user>\week4_etl_pipeline\run_pipeline.py" ^
     /RL HIGHEST
   ```

   Verify it was created:

   ```powershell
   schtasks /Query /TN "Week4 ETL Pipeline" /V /FO LIST
   ```

## What to submit as proof

Replace this section with a screenshot of either:
- `crontab -l` output showing the line above, **or**
- The Windows Task Scheduler "Week4 ETL Pipeline" task properties (General +
  Triggers + Actions tabs), **or**
- The console output of the `schtasks /Query` command above.

Also attach a snippet of `pipeline.log` showing at least one automated,
unattended run (a run with no manual terminal interaction in between the
`Pipeline started` and `Pipeline finished successfully` log lines).
