# server-stats.sh

A simple Bash script to analyse basic performance stats on any Linux server — no non-standard dependencies required.

## What it reports

**Core stats:**
- Total CPU usage (%)
- Total memory usage — free vs. used, including percentages
- Total disk usage — free vs. used, including percentages (per filesystem + overall)
- Top 5 processes by CPU usage
- Top 5 processes by memory usage

**Bonus stats:**
- OS version, kernel, and architecture
- System uptime
- Load average (1 / 5 / 15 min)
- Currently logged-in users
- Recent failed login attempts (requires root)

## Requirements

Works out of the box on most Linux distributions. It only uses standard, pre-installed tools:

- `/proc/stat`, `/proc/loadavg`, `/proc/cpuinfo`
- `free`
- `df`
- `ps`
- `uname`, `uptime`, `who`, `lastb`

No installation of extra packages is needed.

## Usage

1. Copy `server-stats.sh` to the server you want to analyse.
2. Make it executable:
   ```bash
   chmod +x server-stats.sh
   ```
3. Run it:
   ```bash
   ./server-stats.sh
   ```

### Running with root privileges

The **failed login attempts** section reads `/var/log/btmp`, which normally requires elevated permissions. Run with `sudo` to include it:

```bash
sudo ./server-stats.sh
```

Without root, the script still runs fully — it just skips that one section with a short note.

## Example output

```
Server Performance Stats — Mon Sep 14 12:30:58 UTC 2026
Host: myserver

== Total CPU Usage ==
CPU Usage: 9.9% used  (90.1% idle)
CPU Cores: 4

== Memory Usage ==
Total: 4000 MB
Used:  209 MB  (5.2%)
Free:  3867 MB  (96.7%)
Available (for new apps): 3790 MB

== Disk Usage (all mounted filesystems) ==
Filesystem           Size     Used     Avail    Use%   Mounted on
/dev/vda              252G    8.6G     10G      47%    /

Overall: Size=252G Used=8.6G (47%) Avail=10G

== Top 5 Processes by CPU Usage ==
PID      USER                 %CPU   %MEM   COMMAND
1234     root                 7.6    0.1    nginx
...

== Top 5 Processes by Memory Usage ==
PID      USER                 %CPU   %MEM   COMMAND
5678     root                 5.1    0.8    mysqld
...

== OS Version ==
Ubuntu 24.04.4 LTS
Kernel: 6.8.0-generic
Architecture: x86_64

== Uptime ==
up 3 days, 4 hours, 12 minutes

== Load Average (1 / 5 / 15 min) ==
0.15 / 0.22 / 0.18

== Logged In Users ==
alice on pts/0 since 2026-09-14 09:12
Total sessions: 1

== Failed Login Attempts (recent) ==
Run as root/sudo to view failed login attempts (btmp requires elevated privileges).

Done.
```

## How it works

- **CPU usage** is calculated by taking two snapshots of `/proc/stat` one second apart and computing the percentage of non-idle time between them.
- **Memory** and **disk** usage use `free` and `df` respectively, with pseudo-filesystems (`tmpfs`, `devtmpfs`, `overlay`, `squashfs`) excluded from the disk report to avoid noise.
- **Top processes** use `ps --sort=-%cpu` / `ps --sort=-%mem` to rank all running processes.

## Notes

- Colored/bold output uses `tput` where available and degrades gracefully on terminals that don't support it.
- The script is self-contained — a single file, no config needed.
