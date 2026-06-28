# Lab 01 — Basic Commands & Navigation

## Objective

Practice core navigation and file commands: moving around, creating a directory tree, copying/moving/renaming files, viewing content, and searching.

## Scenario

You've just logged into a fresh server. You need to set up a small project structure, populate it, and inspect it — the everyday first steps on any Linux machine.

## Prerequisites

- A Linux environment (WSL/VM/cloud — Module 01).
- Modules 02 and 03 read.

## Steps & Commands

```bash
# 1. Where am I and who am I?
pwd
whoami

# 2. Create a working area in your home directory
cd ~
mkdir -p lab01/project/{src,logs,config}
cd lab01/project
ls -R

# 3. Create some files
touch src/app.py config/settings.conf logs/app.log
echo "name=demo" > config/settings.conf
printf "line1\nline2\nline3\n" > logs/app.log

# 4. View files
cat config/settings.conf
head -n 2 logs/app.log
tail -n 1 logs/app.log

# 5. Copy, move, rename
cp config/settings.conf config/settings.bak
mv src/app.py src/main.py
ls -lah src config

# 6. Search
find . -name "*.conf"
grep -rn "demo" .

# 7. Inspect tree (install tree if needed)
tree . 2>/dev/null || ls -R
```

## Expected Output (samples)

```
$ ls -R
.:
config  logs  src
./config:
settings.conf
...
$ cat config/settings.conf
name=demo
$ grep -rn "demo" .
./config/settings.conf:1:name=demo
```

## Explanation

- `mkdir -p lab01/project/{src,logs,config}` → brace expansion creates three subdirectories at once; `-p` creates parents.
- `ls -R` → recursive listing of the whole tree.
- `head`/`tail` → first/last lines (Module 03).
- `cp`/`mv` → copy and rename (Module 03).
- `find -name` → locate files by pattern; `grep -rn` → find text with file+line numbers.

## Validation

- `lab01/project` contains `src/main.py`, `config/settings.conf`, `config/settings.bak`, `logs/app.log`.
- `grep -rn "demo" .` returns the settings line.

## Cleanup

```bash
cd ~
rm -r lab01
```

## Troubleshooting

- **`tree: command not found`** → `sudo apt install tree` or rely on `ls -R`.
- **"No such file or directory"** → confirm you're in `~/lab01/project` with `pwd`.
- **Permission denied** → ensure you're working under your home directory.
