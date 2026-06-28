# Shell Script Basics

## 1. What Is This?

A **shell script** is a text file containing commands the shell runs top to bottom. It automates what you'd otherwise type by hand.

## 2. Why Is This Needed?

Manual steps are slow and error-prone. A script does the same thing perfectly every time, can be shared, version-controlled, and scheduled.

## 3. Simple Layman Explanation

A script is a **recipe card**: write the steps once, then anyone (or a scheduler) can follow them exactly without remembering each step.

## 4. Technical Explanation

- The first line, the **shebang** `#!/bin/bash`, tells the system which interpreter to use.
- The file is plain text; you make it **executable** with `chmod +x`.
- Run it with `./script.sh` (needs `+x`) or `bash script.sh` (no `+x` needed).
- Comments start with `#`. Exit status `0` = success, non-zero = failure (`echo $?` to check).

## 5. Real-World Example

Instead of typing five commands to deploy, you run `./deploy.sh`. New team members run the same script and get identical results — no tribal knowledge required.

## 6. Diagram

```mermaid
flowchart LR
    Write[Write script.sh] --> Shebang[#!/bin/bash]
    Shebang --> Chmod[chmod +x]
    Chmod --> Run[./script.sh]
    Run --> Out[Output / exit code]
```

## 7. Commands

A first script — save as `hello.sh`:

```bash
#!/bin/bash
# A simple first script

set -euo pipefail            # safe mode: exit on error/undefined var

echo "Hello, $(whoami)!"     # greet the current user
echo "Today is $(date +%F)"  # print the date
echo "You are in: $(pwd)"    # print current directory
```

Run it:

```bash
chmod +x hello.sh        # make it executable
./hello.sh               # run it
bash hello.sh            # alternative (no +x needed)
echo $?                  # exit code of last command (0 = success)
```

## 8. Command Explanation

- `#!/bin/bash` → run this file with bash.
- `set -euo pipefail` → **safe header**: `-e` exit on any error, `-u` error on unset variables, `-o pipefail` catch failures in pipelines.
- `$(command)` → **command substitution**: runs the command and inserts its output.
- `chmod +x` → grants execute permission.
- `echo $?` → shows the exit status of the previous command.

Expected output:

```
Hello, alice!
Today is 2026-06-28
You are in: /home/alice
```

## 9. Practice Tasks

1. Create and run `hello.sh` above.
2. Add a line printing your hostname (`hostname`).
3. Run it with both `./hello.sh` and `bash hello.sh`.
4. Break a command on purpose and observe `set -e` stopping the script.

## 10. Common Mistakes

- Missing shebang → it may run with the wrong shell.
- Forgetting `chmod +x`, then "Permission denied".
- Editing on Windows and saving CRLF line endings → "bad interpreter" errors.

## 11. Troubleshooting

- **"Permission denied"** → `chmod +x script.sh` (or use `bash script.sh`).
- **"bad interpreter: /bin/bash^M"** → CRLF line endings; fix with `dos2unix script.sh`.
- **"command not found"** for the script → run `./script.sh` (with `./`), not `script.sh`.

## 12. Best Practices

- Start every script with `#!/bin/bash` and `set -euo pipefail`.
- Comment the purpose at the top.
- Keep scripts in a known directory; make them executable.

## 13. Quick Recap

- Shebang + `chmod +x` + `./script.sh`.
- `set -euo pipefail` makes scripts fail safely.
- `$(...)` substitutes command output; `$?` is the exit code.

## 14. References

- GNU Bash manual: https://www.gnu.org/software/bash/manual/
- `man bash`

<!-- NAV-FOOTER -->

---

### 🧭 Navigation

| Previous | Up | Next |
|:---|:---:|---:|
| ⬅️ Prev: [Module 10 — Shell Scripting](README.md) | ⬆️ Module: [Module 10 — Shell Scripting](README.md) | ➡️ Next: [Variables, Conditions, and Loops](variables-conditions-loops.md) |
