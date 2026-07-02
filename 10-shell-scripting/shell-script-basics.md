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

## 5. How It Works Under the Hood

A script is "just commands in a file," but three mechanisms decide how it runs — and explain every beginner error:

- **The shebang picks the interpreter — but only via `./script.sh`.** When you run `./script.sh`, the kernel reads the first two bytes; if they're `#!`, it runs the named interpreter (`/bin/bash`) *with your file as input*. So the shebang matters when the file is executed directly. But `bash script.sh` ignores the shebang entirely (you named the interpreter yourself) — which is why a script with no `+x` bit still runs via `bash script.sh`. This is the same shell-vs-shell issue from [Kernel, Shell & Terminal](../02-linux-basics/kernel-shell-terminal.md): omit the shebang and cron/another shell may run it with `sh`, breaking bash-only features.
- **Execution is sequential with an exit code per command.** The shell runs line by line; each command returns an **exit status** (0 = success) the next step can check with `$?` (from [Terminal Basics](../01-linux-setup/terminal-basics.md)). By default the script *keeps going even after a failure* — which is dangerous, and why `set -e` exists (below).
- **`set -euo pipefail` changes the failure model.** By default a script marches past errors. `-e` aborts on any command that fails, `-u` treats using an *unset* variable as an error (catching typos and the infamous empty-path bug), and `-o pipefail` makes a pipeline fail if *any* stage fails (not just the last). Together they turn "silently do the wrong thing" into "stop immediately" — the single most important safety habit in scripting.
- **The CRLF trap:** the kernel looks for `#!/bin/bash` exactly. If a Windows editor saved the file with `\r\n` line endings, the shebang becomes `#!/bin/bash\r`, and the kernel tries to run an interpreter literally named `/bin/bash^M` → `bad interpreter`. That's why line endings matter.

## 6. Diagram

```mermaid
flowchart LR
    Write[Write script.sh] --> Shebang["#!/bin/bash (kernel reads on ./run)"]
    Shebang --> Safe["set -euo pipefail (fail fast)"]
    Safe --> Chmod[chmod +x]
    Chmod --> Run[./script.sh]
    Run --> Out[Output + exit code $?]
```

## 7. Real-World Examples

**1. The everyday case.** Instead of typing five commands to deploy, you run `./deploy.sh`. New team members run the same script and get identical results — no tribal knowledge required.

**2. A first script and its output:**

```
$ cat hello.sh
#!/bin/bash
set -euo pipefail
echo "Hello, $(whoami)!"
echo "Today is $(date +%F)"
echo "You are in: $(pwd)"
$ chmod +x hello.sh
$ ./hello.sh
Hello, alice!
Today is 2026-07-02
You are in: /home/alice
$ echo $?
0                              # success
```

`$(...)` ran each command and inserted its output; the final `$?` of 0 confirms success (Section 5).

**3. War story — the deploy that "worked" while failing.** A deploy script without `set -e` did: `cd /app/build`, then `npm run build`, then `cp -r dist/* /var/www/`. One day `cd /app/build` failed (the dir was renamed), but the script *kept going* — running the build and copy from the wrong directory, publishing stale files. It exited `0`, so CI reported success while production served old code. Adding `set -euo pipefail` (and guarding `cd ... || exit 1`) made it abort at the failed `cd` instead. Without fail-fast, a script's "success" means nothing.

## 8. Worked Walkthrough

Create a script, make it safe, and watch fail-fast work:

```
$ cat > demo.sh <<'EOF'
#!/bin/bash
set -euo pipefail
echo "step 1 ok"
cp /does/not/exist /tmp/   # this FAILS
echo "step 2 (should NOT print)"
EOF
$ chmod +x demo.sh
$ ./demo.sh
step 1 ok
cp: cannot stat '/does/not/exist': No such file or directory
$ echo $?
1                              # aborted at the failing line — step 2 never ran
```

Now remove `set -euo pipefail` and re-run: you'll see `step 2 (should NOT print)` appear despite the error — exactly the silent-continue bug from the war story (Section 5). That contrast is why the safety header is non-negotiable.

## 9. Commands

```bash
# A first script — save as hello.sh:
#!/bin/bash
set -euo pipefail            # safe mode: exit on error/undefined var/pipe failure
echo "Hello, $(whoami)!"     # greet the current user
echo "Today is $(date +%F)"  # print the date
echo "You are in: $(pwd)"    # print current directory

# Run/verify it:
chmod +x hello.sh            # make it executable
./hello.sh                   # run it (kernel reads the shebang)
bash hello.sh                # alternative (no +x needed; shebang ignored)
echo $?                      # exit code of last command (0 = success)
bash -n hello.sh             # syntax-check without running
```

Sample output for each (dummy values, for reference):

```text
$ ./hello.sh
Hello, alice!
Today is 2026-07-02
You are in: /home/alice

$ echo $?
0

$ bash hello.sh
Hello, alice!
Today is 2026-07-02
You are in: /home/alice

$ bash -n hello.sh
# (no output = syntax OK)
```

## 10. Command Explanation

- `#!/bin/bash` → run this file with bash (read by the kernel on `./script.sh`).
- `set -euo pipefail` → **safe header**: `-e` exit on any error, `-u` error on unset variables, `-o pipefail` catch failures anywhere in a pipeline (Section 5).
- `$(command)` → **command substitution**: runs the command and inserts its output.
- `chmod +x` → grants execute permission (needed for `./script.sh`).
- `bash -n` → checks syntax without executing; `echo $?` → shows the previous command's exit status.

## 11. In Production (DevOps Context)

- **Everything is a script eventually:** deploys, health checks, backups, and CI/CD steps are shell scripts on Linux runners (Module 13).
- **`set -euo pipefail` is a CI reliability feature:** without fail-fast, a broken step can exit `0` and let a red pipeline look green (the war story) — masking failures until production.
- **Shebang + line endings matter in automation:** cron/CI may default to `sh`; a missing shebang or CRLF endings cause "works on my machine, fails in CI" (Modules 02, 11).
- **Version control + review:** scripts live in Git so changes are reviewed and auditable — turning tribal commands into shared, tested tooling.

## 12. Practice Tasks

1. Create and run `hello.sh` above; check `echo $?`.
2. Add a line printing your hostname (`hostname`).
3. Run it with both `./hello.sh` and `bash hello.sh`.
4. Reproduce the war story: add a failing command and observe `set -e` stopping the script (then remove `set -e` and see it continue).

## 13. Common Mistakes

- Omitting `set -euo pipefail`, so errors pass silently (the war story).
- Missing shebang → it may run with the wrong shell (`sh` in cron/CI).
- Forgetting `chmod +x`, then "Permission denied" (use `bash script.sh` or add `+x`).
- Editing on Windows and saving CRLF line endings → "bad interpreter" errors (Section 5).

## 14. Troubleshooting

- **"Permission denied"** → `chmod +x script.sh` (or run `bash script.sh`).
- **"bad interpreter: /bin/bash^M"** → CRLF line endings; fix with `dos2unix script.sh`.
- **"command not found" for the script** → run `./script.sh` (with `./`), not `script.sh`.
- **Script continues after an error** → you're missing `set -e`; add the safety header.

## 15. Best Practices

- Start every script with `#!/bin/bash` and `set -euo pipefail`.
- Comment the purpose at the top; keep scripts in Git.
- Guard risky steps (`cd ... || exit 1`); syntax-check with `bash -n` before shipping.

## 16. Connects To

- **Prev:** [Module 10 — Shell Scripting](README.md). **Next:** [Variables, Conditions & Loops](variables-conditions-loops.md).
- **Shells & shebangs:** [Kernel, Shell & Terminal](../02-linux-basics/kernel-shell-terminal.md); **exit codes:** [Terminal Basics](../01-linux-setup/terminal-basics.md).
- **Making scripts safe/runnable:** [Script Permissions & Safety](script-permissions.md).
- **Scheduling them:** [Module 11 — Automation & Cron](../11-automation-and-cron/README.md).

## 17. Quick Recap

- Shebang (read by the kernel on `./script.sh`) + `chmod +x` + `./script.sh`.
- `set -euo pipefail` flips the model from "continue past errors" to "fail fast" — the key safety habit.
- `$(...)` substitutes command output; `$?` is the exit code; CRLF endings break the shebang.

## 18. References

- GNU Bash manual: https://www.gnu.org/software/bash/manual/
- `man bash`

<!-- NAV-FOOTER -->

---

### 🧭 Navigation

| Previous | Up | Next |
|:---|:---:|---:|
| ⬅️ Prev: [Module 10 — Shell Scripting](README.md) | ⬆️ Module: [Module 10 — Shell Scripting](README.md) | ➡️ Next: [Variables, Conditions, and Loops](variables-conditions-loops.md) |
