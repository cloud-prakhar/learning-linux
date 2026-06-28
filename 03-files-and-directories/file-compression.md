# File Compression & Archiving

## 1. What Is This?

**Archiving** bundles many files into one (`tar`). **Compression** shrinks size (`gzip`, `bzip2`, `zip`). Often combined: `tar.gz` = archived **and** compressed.

## 2. Why Is This Needed?

You move and store lots of files: backups, logs, transfers. Bundling + compressing saves space and makes transfers a single, smaller download.

## 3. Simple Layman Explanation

- `tar` = put many documents into one envelope.
- `gzip` = vacuum-seal the envelope to make it smaller.
- `tar.gz` = both: one sealed, compact package.

## 4. Technical Explanation

| Tool | Role | Common Use |
|------|------|-----------|
| `tar` | Archive multiple files into one | `.tar` |
| `gzip` | Compress a single file | `.gz` |
| `tar + gzip` | Archive + compress | `.tar.gz` / `.tgz` |
| `zip`/`unzip` | Cross-platform archive+compress | `.zip` (Windows-friendly) |

## 5. Real-World Example

To send logs for analysis: `tar -czf logs-$(date +%F).tar.gz /var/log/myapp/`. One compact file, timestamped, easy to upload. This is the standard way ops engineers share logs.

## 6. Diagram

```mermaid
flowchart LR
    Files[Many files] --> Tar[tar: bundle into one]
    Tar --> Gz[gzip: compress]
    Gz --> Archive[(archive.tar.gz)]
    Archive --> Extract[tar -xzf: extract back]
```

## 7. Commands

```bash
tar -czf backup.tar.gz mydir/      # create gzipped archive
tar -xzf backup.tar.gz             # extract gzipped archive
tar -tzf backup.tar.gz             # list contents without extracting
tar -czf logs.tar.gz /var/log/app  # archive a directory
gzip bigfile.log                   # compress -> bigfile.log.gz
gunzip bigfile.log.gz              # decompress
zip -r project.zip project/        # create a zip
unzip project.zip                  # extract a zip
```

## 8. Command Explanation

`tar` flags (memorize this combo):
- `-c` = **c**reate an archive
- `-x` = e**x**tract
- `-t` = lis**t** contents
- `-z` = gzip compression (`-j` for bzip2)
- `-f` = **f**ile name (must come right before the filename)
- `-v` = verbose (show files as they process)

So `tar -czvf out.tar.gz dir/` = create + gzip + verbose + file `out.tar.gz` from `dir/`.

## 9. Practice Tasks

1. `mkdir demo && touch demo/a.txt demo/b.txt`.
2. `tar -czf demo.tar.gz demo/`.
3. `tar -tzf demo.tar.gz` to list contents.
4. `rm -r demo && tar -xzf demo.tar.gz` to restore it.
5. `zip -r demo.zip demo/` and `unzip -l demo.zip`.

## 10. Common Mistakes

- Forgetting `-f` or putting it in the wrong place. Keep `-f` last before the filename.
- Mixing up `-c` (create) and `-x` (extract) — read carefully.
- Extracting an archive that has no top folder, scattering files into the current dir. Check first with `-t`.

## 11. Troubleshooting

- **"tar: Cannot open: No such file"** → wrong filename/path after `-f`.
- **`gzip: already has .gz suffix`** → it's already compressed.
- **`unzip: command not found`** → install it (`sudo apt install unzip`).

## 12. Best Practices

- Always **list** (`-tzf`) an unknown archive before extracting.
- Timestamp backups: `backup-$(date +%F).tar.gz`.
- Use `.tar.gz` for Linux-to-Linux, `.zip` when Windows users are involved.

## 13. Quick Recap

- `tar -czf x.tar.gz dir/` create, `-xzf` extract, `-tzf` list.
- `gzip`/`gunzip` for single files; `zip`/`unzip` for cross-platform.
- Remember c/x/t + z + f.

## 14. References

- GNU Tar manual: https://www.gnu.org/software/tar/manual/
- `man tar`, `man gzip`, `man zip`
