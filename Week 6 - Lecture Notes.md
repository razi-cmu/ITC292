# Week 6: Managing Backups and System Logs

System administration is not just about keeping systems running but it's also about being prepared for when things go wrong. Two of the most critical responsibilities of a system administrator are ensuring that data can be recovered after a failure and being able to understand what happened when issues occur. This week covers two foundational pillars of system administration: backups (protecting data) and system logs (understanding system behavior).

The Linux kernel itself treats many things as mere numbers such as processes, users, and even log messages have numeric priorities. But as system administrators, we work with these abstractions to build reliable, observable systems.


## Managing Backups

As they commonly say: Hardware is indeterministically reliable. Software is deterministically unreliable. People are indeterministically unreliable. Nature is deterministically reliable.

Data loss can happen for many reasons:
- Hardware failure: disk crashes, power surges, aging components
- Human error: accidental deletion, misconfiguration
- Software bugs: corruption, data loss from application errors
- Security incidents: ransomware, intrusion, data breaches
- Natural disasters: fire, flood, theft

The fundamental principle of backups is simple: A backup is only useful if you can successfully restore from it.


### What to Back Up?
Not all data is equally important. Prioritize your backups:

| Priority | Data Type | Examples |
|----------|-----------|----------|
| Critical | User data | `/home/`, databases, application data |
| High | System configuration | `/etc/`, `/boot/`, custom scripts |
| Medium | System logs | `/var/log/` (for auditing and troubleshooting) |
| Low | Cached/temporary data | `/tmp/`, `/var/cache/` (can be excluded) |

What NOT to Back Up:

| Path | Reason to Exclude |
|------|-------------------|
| `/proc/` | Virtual filesystem containing kernel data structures. |
| `/dev/` | Device files that are recreated automatically at boot. |
| `/tmp/` | Temporary files that do not need to be backed up. |
| `/var/run/` | Runtime data generated during system operation. |


### Types of Backups
There are three primary backup strategies:

#### Full Backup
A complete copy of all selected data at a specific point in time.

Characteristics:
- ✅ Simplest to restore (single archive)
- ✅ Complete data set
- ❌ Largest storage requirements
- ❌ Slowest to create
- ❌ Most network bandwidth consumed

Common Tools: `tar`, `rsync`, `dd`, `dump`

#### Incremental Backup
Backs up only data that has changed since the most recent backup (full or incremental).

Characteristics:
- ✅ Smallest storage footprint
- ✅ Fastest to create
- ✅ Minimal bandwidth usage
- ❌ Complex restore process (must restore full + all increments in order)
- ❌ If one incremental is corrupted, later ones may be unusable

#### Differential Backup
Backs up all data that has changed since the last full backup.

Characteristics:
- ✅ Easier restore than incremental (full + latest differential only)
- ✅ Larger than incremental, smaller than full
- ❌ Grows larger each day until next full backup
- ❌ More storage than incremental



### Comparison
| Feature | Full | Incremental | Differential |
|---------|------|-------------|---------------|
| Storage space | Most | Least | Medium |
| Backup speed | Slowest | Fastest | Medium |
| Restore speed | Fastest | Slowest | Medium |
| Complexity | Simple | Complex | Moderate |
| Risk | Low | Higher (chain dependency) | Medium |

>**Recommendation**: Full backup weekly + Incremental daily (or Differential daily + Full weekly)

### Backup Tools Overview

| Tool        | Primary Use              | Key Feature                                                                 |
|-------------|--------------------------|------------------------------------------------------------------------------|
| `tar`       | Archive creation         | Combines files into single archive; supports compression                   |
| `rsync`       | Synchronization          | Copies only differences; supports remote sync                               |
| `dd`          | Disk/partition cloning   | Bit-level copy of entire devices                                            |
| `dump/restore` | Filesystem backup        | Traditional UNIX backup utilities                                            |

### Basic Backup with tar
The `tar` command (Tape ARchive) is one of the most common tools for creating backups.

Creating a backup:

```bash
# Create a compressed archive of test directory
tar -czf test-$(date +%Y%m%d).tar.gz test
```

In the above command, `tar` will compress a test folder into `test-20260701.tar.gz` where `20260701` is the date and `tar.gz` is the file extension as Linux using `gzip` to compress files. Furthermore, `c` is for starting a new archive (compressed file), `z` is for zipping the file and `f` means a file name will be provided for compression.

Once a file is compressed or backed up, it can be restored using `tar` as below:

```bash
# Extract the entire archive
tar -xzf test-20260701.tar.gz -C ~/Desktop
```

In the above command `x` is to extract. The above command extracts the test folder from the archive on Desktop.

There might be instances where only certain files need to be extracted from the archieve. This can be achieved as below:

```bash
# Extract only specific files
tar -xzf test-20260701.tar.gz test/test.txt
```

The above command will extract `test.txt` from the test folder in `test-20260701.tar.gz`.

### Backup Automation Script Template
A well-structured backup script typically follows this pattern:

- Define variables: source directories, destination, date stamp
- Create backup: using tar, rsync, or other tools
- Compress (if using tar)
- Verify: check exit code, file size
- Log results: record success/failure
- Cleanup: remove old backups (retention policy)
- Notify: send alerts on failure

## System Logs
System logs are records of events that occur within the operating system and its applications. The `syslog` service is a fundamental part of any UNIX system.

Purposes:
- Troubleshooting: diagnosing problems and errors
- Monitoring: tracking system health and performance
- Security auditing: detecting unauthorized access or anomalies
- Compliance: meeting regulatory requirements
- Forensics: investigating incidents after they occur


### Syslog Severity Levels
| Level | Name   | Description                          |
|-------|--------|--------------------------------------|
| 0     | emerg  | System is unusable                  |
| 1     | alert  | Action must be taken immediately     |
| 2     | crit   | Critical conditions                  |
| 3     | err    | Error conditions                     |
| 4     | warn   | Warning conditions                   |
| 5     | notice | Normal but significant conditions    |
| 6     | info   | Informational messages               |
| 7     | debug  | Debugging-level messages             |

### Log File Locations
Common Log Files in `/var/log/`:

| File                          | Purpose                          |
|-------------------------------|----------------------------------|
| /var/log/syslog or /var/log/messages | General system messages |
| /var/log/auth.log            | Authentication and security events |
| /var/log/kern.log            | Kernel messages                  |
| /var/log/daemon.log          | System daemon messages           |
| /var/log/mail.log            | Mail server logs                |
| /var/log/cron.log            | cron job logs                   |
| /var/log/boot.log            | Boot process messages           |
| /var/log/dmesg               | Kernel ring buffer messages     |

### Modern Logging Systems
Two Main Components in Modern Linux are used for logging:

#### systemd-journald
It collects logs from the kernel, system services, and applications. It stores logs in binary format (indexed for fast searching). It is normally integrated with `systemd`. 

#### rsyslog
It is traditional syslog implementation and reads messages and writes them to log files. It supports filtering, forwarding, and output to various destinations.

Configuration: `/etc/rsyslog.conf and /etc/rsyslog.d/*.conf`

Coexistence: journald collects, rsyslog writes to files.


### Viewing Logs using `journalctl`
`journalctl` is the primary command for viewing journald logs.

Common Usage:

```bash
journalctl                                       # View all logs
journalctl -f                                    # Follow (tail) logs in real-time
journalctl -n 20                                 # Show last 20 lines
journalctl -u service-name                       # Filter by service unit
journalctl --since "1 hour ago"                  # Time-based filtering
journalctl -b                                    # Current boot only
journalctl -b -1                                 # Previous boot

journalctl --vacuum-size=500M                    # Limit journal size
journalctl --vacuum-time=30d                     # Keep only last 30 days

## Viewing Logs — Traditional Commands

less /var/log/syslog                             # View with pagination
tail -f /var/log/syslog                          # Follow log in real-time
grep "error" /var/log/syslog                     # Search for specific terms
tail -f /var/log/syslog | grep -i "fail"         # Monitor for failures only
```


## Exercise
Write a Bash script that automates the backup of a specific directory and implements a cleanup policy to manage disk space.

### Requirements:
- Define Variables: At the top of your script, define variables for the `SOURCE_DIR` (e.g., `~/Desktop/test`), the `BACKUP_DIR` (e.g., `~/Documents/temp`), and a `RETENTION_DAYS` variable set to `30`.
- Date Formatting: Use the date command to include the current date in the filename (e.g., `backup-20260701.tar.gz`).
- Perform Backup: Write the command to compress the `SOURCE_DIR` into the `BACKUP_DIR` using `tar`. Redirect any potential error output to `/dev/null`.
- Conditional Check: Use an `if` statement to check if the `tar` command succeeded.
- Cleanup Logic: If the backup was successful, use the find command to automatically delete any files inside the `BACKUP_DIR` that match the backup naming pattern and are older than your `RETENTION_DAYS` value.
- Error Feedback: If the backup fails, output an error message to Standard Error `(stderr)` and ensure the script exits with a non-zero status code.

### Solution

```bash
#!/bin/bash

# A backup cleaning script

BACKUP_DIR="$HOME/Documents/temp"
SOURCE_DIR="$HOME/Desktop/test"
DATE=$(date +%Y%m%d)
RETENTION_DAYS=30

tar -czf "$BACKUP_DIR/backup-$DATE.tar.gz" "$SOURCE_DIR" 2>/dev/null

if [ $? -eq 0 ]; then
	echo "Backup was successful: backup-$DATE.tar.gz"
	find "$BACKUP_DIR" -name "backup-*.tar.gz" -mtime +$RETENTION_DAYS -delete
else
	echo "Error: Backup failed!" >&2
	exit 1
fi
```

## Ungraded Exercises
You can perform these exercises on any Linux system (virtual machine, WSL, or a real Linux installation). Root access is not required for most commands, except for installing tools. Use only scripts or commands to complete these exercises.

### Exercise 1:
Create a simple backup script that backs up your home directory. Feel free to add variations to it, e.g., create a backup directory at your `root`directory. Start by doing it with multiple terminal commands and then just write a bash script to achieve the same results. 

>Hint: You can use `$HOME` in your script to refer to the home directory.

### Exercise 2:
Schedule the backup script you have created in Exercise 1 to run daily at 2:00 AM. You can follow below steps assuming you already have the backup script from Exercise 1 in a file named `backup.sh`

- Edit your crontab:

```bash
crontab -e
```

- Add scheduling rule:
```bash
0 2 * * * /home/yourusername/backup_home.sh >> /home/yourusername/backup.log 2>&1
```
Review man pages for crontab to learn about the syntax. This is the actual Exercise 2, e.g., to learn about running scheduled rules.

- Verify the cron job is scheduled:

```bash
crontab -l
```

- Check that the log file is being created:

```bash
cat ~/backup.log
```

### Exercise 3:
Write a Bash script that performs basic log cleanup:

- Compress any file in `/var/log` that is larger than 50MB (use `gzip`).
- Move any file in `/var/log` that is older than 7 days to an archive folder `/var/log/archive`.
- Print a one line summary showing how many files were compressed and how many were moved.

#### Requirements:
- Use variables for the size limit, age limit, and directory paths.
- Create the archive folder if it does not already exist.
- If a file cannot be compressed or moved (e.g., `permission denied`), skip it and continue. Make sure the script does not crash.

>**Hint**: Use `find with -maxdepth 1, -size, -mtime, and ! -name "*.gz"` to skip already compressed files. Feel free to do it differently if you want to.

## References
- The Linux System Administrator's Guide — Chapter 12: Backups
- The Linux System Administrator's Guide — Chapter 15: System Logs
- Manual pages: `man tar`, `man cron`, `man logrotate`, `man journalctl`, `man syslog`
