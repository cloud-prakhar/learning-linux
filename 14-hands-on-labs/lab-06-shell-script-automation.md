# Lab 06 — Shell Script Automation

## Objective

Write a small, safe shell script with arguments and validation, make it executable, run it, then schedule it with cron.

## Scenario

You need a repeatable task automated: a script that snapshots a directory's file count and disk usage into a dated report, run on a schedule.

## Prerequisites

- A Linux environment.
- Modules 10 and 11 read.

## Steps & Commands

```bash
# 1. Create the script
mkdir -p ~/lab06/reports
cat > ~/lab06/dirreport.sh <<'EOF'
#!/bin/bash
# dirreport.sh <target-dir> <report-dir>
# Writes a dated report of file count and size for target-dir.
set -euo pipefail

TARGET="${1:-}"
REPORT_DIR="${2:-}"

if [ -z "$TARGET" ] || [ -z "$REPORT_DIR" ]; then
    echo "Usage: $0 <target-dir> <report-dir>" >&2
    exit 1
fi
if [ ! -d "$TARGET" ]; then
    echo "Error: '$TARGET' is not a directory" >&2
    exit 1
fi

mkdir -p "$REPORT_DIR"
STAMP="$(date +%F_%H%M%S)"
REPORT="$REPORT_DIR/report-$STAMP.txt"

{
  echo "Report for: $TARGET"
  echo "Generated:  $(date)"
  echo "File count: $(find "$TARGET" -type f | wc -l)"
  echo "Total size: $(du -sh "$TARGET" | cut -f1)"
} > "$REPORT"

echo "Wrote $REPORT"
EOF

# 2. Make it executable and inspect
chmod +x ~/lab06/dirreport.sh
ls -l ~/lab06/dirreport.sh

# 3. Run it (and test validation)
~/lab06/dirreport.sh                 # should print usage and exit 1
~/lab06/dirreport.sh /etc ~/lab06/reports
cat ~/lab06/reports/report-*.txt

# 4. Debug-trace it once
bash -x ~/lab06/dirreport.sh /etc ~/lab06/reports >/dev/null

# 5. Schedule it every 2 minutes (temporary), with logging
( crontab -l 2>/dev/null; \
  echo "*/2 * * * * $HOME/lab06/dirreport.sh /etc $HOME/lab06/reports >> $HOME/lab06/cron.log 2>&1" ) | crontab -
crontab -l

# 6. Wait ~2-3 minutes, then verify cron ran it
sleep 130
ls -1 ~/lab06/reports/
tail -n 5 ~/lab06/cron.log
```

## Expected Output (samples)

```
$ ~/lab06/dirreport.sh
Usage: /home/you/lab06/dirreport.sh <target-dir> <report-dir>

$ cat ~/lab06/reports/report-*.txt
Report for: /etc
Generated:  Sat Jun 28 10:00:00 2026
File count: 312
Total size: 4.7M
```

## Explanation

- `cat > file <<'EOF' ... EOF` → a **heredoc**; quoting `'EOF'` prevents variable expansion so the script is written literally.
- `set -euo pipefail` + argument validation → safe scripting (Module 10).
- `chmod +x` → makes it runnable; running with no args prints usage and exits 1.
- `bash -x` → traces each line for debugging.
- The cron line uses **absolute paths** (`$HOME/...`) and `>> log 2>&1` — the Module 11 essentials.
- `( crontab -l; echo "...") | crontab -` → appends a job without opening an editor.

## Validation

- Running with no args prints usage (exit 1); with args it writes a dated report.
- After ~2 minutes, new `report-*.txt` files appear and `cron.log` shows "Wrote ...".

## Cleanup

```bash
# Remove the cron job we added
crontab -l | grep -v 'lab06/dirreport.sh' | crontab -
crontab -l
# Remove the lab files
rm -rf ~/lab06
```

## Troubleshooting

- **Script "Permission denied"** → `chmod +x` (step 2) or run `bash ~/lab06/dirreport.sh`.
- **Cron job didn't run** → check `~/lab06/cron.log`; ensure absolute paths and that the cron daemon is active (`systemctl status cron`). See [cron-troubleshooting](../11-automation-and-cron/cron-troubleshooting.md).
- **`crontab: command not found`** → install cron (`sudo apt install cron`) and start it.
- **CRLF / "bad interpreter"** → if edited on Windows, run `dos2unix ~/lab06/dirreport.sh`.
