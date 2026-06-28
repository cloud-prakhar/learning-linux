# Mermaid Diagrams — Visual Reference

All key diagrams in one place. They render on GitHub automatically.

---

## 1. Linux Architecture (Module 02)

```mermaid
flowchart LR
    User[User] --> Apps[Applications]
    Apps --> Shell[Shell - bash]
    Shell --> Libs[System Libraries]
    Libs --> Kernel[Linux Kernel]
    Kernel --> HW[Hardware: CPU / RAM / Disk / Network]
```

## 2. User ↔ Shell ↔ Kernel (Module 02)

```mermaid
flowchart LR
    Term[Terminal] --> Shell[Shell interprets command]
    Shell --> Kernel[Kernel does the work]
    Kernel --> HW[Hardware]
    Kernel --> Shell
    Shell --> Term
```

## 3. Filesystem Hierarchy (Module 02)

```mermaid
flowchart TD
    Root["/"] --> etc[/etc - configs/]
    Root --> home[/home - users/]
    Root --> var[/var - logs & data/]
    Root --> bin[/bin - commands/]
    Root --> tmp[/tmp - temp/]
    var --> log[/var/log/]
    home --> user[/home/alice/]
```

## 4. Permission Model (Module 04)

```mermaid
flowchart LR
    File["-rwxr-xr--"] --> O[Owner: rwx = 7]
    File --> G[Group: r-x = 5]
    File --> Ot[Others: r-- = 4]
    O & G & Ot --> Num[= 754]
```

## 5. Process Lifecycle (Module 05)

```mermaid
flowchart LR
    New[Start] --> Run[Running]
    Run --> Sleep[Sleeping]
    Sleep --> Run
    Run --> Stop[Stopped]
    Run --> Done[Terminated]
```

## 6. systemd Service Flow (Module 05)

```mermaid
flowchart LR
    Boot[Boot] --> Systemd[systemd PID 1]
    Systemd --> S1[nginx.service]
    Systemd --> S2[ssh.service]
    Systemd --> S3[cron.service]
    Ctl[systemctl] --> Systemd
```

## 7. Package Installation Flow (Module 06)

```mermaid
flowchart LR
    You[apt install nginx] --> PM[Package Manager]
    PM --> Repo[(Repository)]
    Repo --> Deps[Nginx + dependencies]
    Deps --> Sys[Installed on system]
```

## 8. DNS Resolution Flow (Module 07)

```mermaid
flowchart LR
    App[App asks for example.com] --> Hosts{/etc/hosts?}
    Hosts -->|found| IP[Use that IP]
    Hosts -->|not found| DNS[(DNS server)]
    DNS --> IP
```

## 9. Client-Server Connection Flow (Module 07)

```mermaid
flowchart LR
    C[Client/Browser] -->|1. DNS lookup| DNS[(DNS)]
    DNS -->|IP| C
    C -->|2. TCP connect port 443| S[Server]
    S -->|3. HTTP response| C
```

## 10. Disk Mount Flow (Module 08)

```mermaid
flowchart LR
    Disk[/dev/sda - disk/] --> Part[/dev/sda1 - partition/]
    Part --> FS[ext4 filesystem]
    FS --> Mount[mounted at /]
    Mount --> Tree[(directory tree)]
```

## 11. Log Troubleshooting Flow (Module 09)

```mermaid
flowchart TD
    Sym[Symptom + time] --> Res[Check resources: top/free/df]
    Res --> Logs[Check logs: journalctl --since / /var/log]
    Logs --> Hyp[Hypothesis -> test -> fix]
    Hyp --> Prev[Verify -> Prevent]
```

## 12. Cron Job Execution Flow (Module 11)

```mermaid
flowchart LR
    Crontab[crontab entry] --> Cron[cron daemon checks every minute]
    Cron --> Match{Time matches?}
    Match -->|yes| Run[Run command/script]
    Match -->|no| Wait[Wait 1 minute]
```

## 13. SSH Connection Flow (Module 12)

```mermaid
flowchart LR
    Priv[Private key] -->|proves identity| Client[ssh client]
    Client -->|port 22, encrypted| Server[sshd]
    Server --> Auth[checks authorized_keys]
    Auth -->|match| Shell[Logged in]
```

## 14. Linux in DevOps Workflow (Module 13)

```mermaid
flowchart LR
    L[Linux Skills] --> AWS[AWS / EC2]
    L --> Docker[Docker]
    L --> K8s[Kubernetes]
    L --> CICD[CI/CD]
    AWS & Docker & K8s & CICD --> Prod[Production Systems]
```

---

> Edit/preview these at the [Mermaid Live Editor](https://mermaid.live/). Keep diagrams simple and left-to-right for clarity.
