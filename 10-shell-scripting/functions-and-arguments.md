# Functions and Arguments

## 1. What Is This?

**Functions** group commands into reusable named blocks. **Arguments** are inputs passed to a script or function so it can act on different values.

## 2. Why Is This Needed?

Functions stop you repeating code and make scripts readable. Arguments make scripts flexible — one backup script can back up any directory you pass it.

## 3. Simple Layman Explanation

A function is a **named recipe step** you can reuse ("make_coffee"). Arguments are the **ingredients** you hand it ("make_coffee large oat-milk").

## 4. Technical Explanation

- Define: `name() { commands; }`. Call: `name arg1 arg2`.
- Inside, positional parameters: `$1`, `$2`, ... `$@` (all args), `$#` (count), `$0` (script name).
- `return N` sets a function's exit status; `echo` returns text output.
- Script-level args work the same: `./script.sh foo` → `$1` is `foo`.

## 5. Real-World Example

A deploy script defines `log()`, `check_disk()`, and `deploy(env)` functions. You call `./deploy.sh staging` and `$1` ("staging") flows through — clean, reusable, testable.

## 6. Diagram

```mermaid
flowchart LR
    Call["script.sh prod 80"] --> Args["$1=prod  $2=80"]
    Args --> Fn[function uses $1 $2]
    Fn --> Result[output / return code]
```

## 7. Commands

Save as `funcs.sh`:

```bash
#!/bin/bash
set -euo pipefail

# A function that logs with a timestamp
log() {
    echo "[$(date +%T)] $*"      # $* = all arguments as one string
}

# A function taking arguments
greet() {
    local who="$1"               # local keeps it inside the function
    local times="${2:-1}"        # default to 1 if $2 not given
    for ((i=0; i<times; i++)); do
        log "Hello, $who"
    done
}

# --- Script arguments ---
if [ "$#" -lt 1 ]; then
    echo "Usage: $0 <name> [times]"
    exit 1
fi

greet "$1" "${2:-2}"             # pass script args into the function
log "Done. Args received: $#"
```

Run:

```bash
chmod +x funcs.sh
./funcs.sh Alice 3
./funcs.sh            # shows usage and exits 1
```

## 8. Command Explanation

- `name() { ...; }` → defines a function.
- `local var=...` → scopes a variable to the function (prevents leaks).
- `"$1"`, `"$2"` → first/second argument; `"$@"` all args (preserves spacing), `$#` count.
- `"${2:-1}"` → **default value**: use `$2`, or `1` if unset.
- `if [ "$#" -lt 1 ]` → argument validation; `exit 1` signals an error.

## 9. Practice Tasks

1. Run `funcs.sh` with and without arguments.
2. Add a function `sum() { echo $(( $1 + $2 )); }` and call `sum 3 4`.
3. Use `"$@"` in a function that prints each argument on its own line.
4. Add a default for a missing argument with `${1:-default}`.

## 10. Common Mistakes

- Forgetting `local`, so function variables overwrite outer ones.
- Not validating `$#`, then the script fails on missing input.
- Confusing `$*` (all args as one word, mostly) with `"$@"` (each arg preserved) — prefer `"$@"`.

## 11. Troubleshooting

- **"$1: unbound variable"** (with `set -u`) → guard with `${1:-}` or check `$#`.
- Function not running → ensure it's **defined before** it's called.
- Wrong values → trace with `bash -x script.sh`.

## 12. Best Practices

- Validate arguments early; print a `Usage:` line.
- Use `local` for function variables.
- Provide sensible defaults with `${var:-default}`.
- Prefer `"$@"` for passing arguments through.

## 13. Quick Recap

- `name() { ...; }`; call with args; access via `$1 $2 "$@" $#`.
- `local` scopes variables; `${2:-default}` sets defaults.
- Validate `$#` and show usage.

## 14. References

- Bash functions: https://www.gnu.org/software/bash/manual/
- `man bash` (Shell Functions, Positional Parameters)
