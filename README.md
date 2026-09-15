# log-archive

A simple command-line tool to archive logs on a schedule — it compresses a log directory into a timestamped `.tar.gz` file and keeps a record of every archive it creates.

Project URL: https://roadmap.sh/projects/log-archive-tool

## What it does

- Takes a log directory as an argument.
- Compresses it into a `.tar.gz` archive.
- Stores the archive in a dedicated archive directory (`~/log_archives` by default).
- Names each archive with a timestamp: `logs_archive_YYYYMMDD_HHMMSS.tar.gz`.
- Appends a line to `archive.log` recording the date/time, source directory, resulting archive path, and size — so you have a running history of every archive that's been made.

## Usage

```bash
log-archive <log-directory>
```

### Example

```bash
log-archive /var/log
```

Output:

```
Logs archived successfully.
  Source:  /var/log
  Archive: /home/user/log_archives/logs_archive_20260914_204633.tar.gz
  Size:    4.0K
  Logged to: /home/user/log_archives/archive.log
```

### Archiving system logs

`/var/log` usually requires elevated permissions to read fully:

```bash
sudo log-archive /var/log
```

### Help

```bash
log-archive --help
```

## Installation

1. Copy the `log-archive` script to your machine.
2. Make it executable:
   ```bash
   chmod +x log-archive
   ```
3. (Optional) Put it on your `PATH` so you can call it from anywhere:
   ```bash
   sudo mv log-archive /usr/local/bin/log-archive
   ```

## Scheduling with cron

To run it automatically (e.g. archive `/var/log` every day at 1 AM), add a crontab entry:

```bash
crontab -e
```

```cron
0 1 * * * /usr/local/bin/log-archive /var/log >> /var/log/log-archive-cron.log 2>&1
```

## Configuration

By default, archives are stored in `~/log_archives`. You can override this with the `LOG_ARCHIVE_DIR` environment variable:

```bash
LOG_ARCHIVE_DIR=/backups/logs log-archive /var/log
```

## Archive record format

Every run appends a line like this to `archive.log` inside the archive directory:

```
[2026-09-14 20:46:33] Archived '/tmp/testlogs' -> '/tmp/fakehome/log_archives/logs_archive_20260914_204633.tar.gz' (size: 4.0K)
```

## How it works

- Resolves the given log directory to an absolute path.
- Uses `tar -czf` to compress it, with paths relative to the parent directory so extracting the archive recreates a folder with the original directory's name (not an absolute-path structure).
- Creates the archive directory (and its `archive.log`) if they don't already exist.
- Validates its input: requires exactly one argument, checks the directory exists and is readable, and reports clear errors otherwise.

## Requirements

Standard Unix tools only — `bash`, `tar`, `date`, `du`. No extra packages needed.
