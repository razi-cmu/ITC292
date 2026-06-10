# Week 4: System Monitoring and Managing Boots & Shutdowns

## Introduction to System Monitoring

### Why Monitor a Linux System?

System monitoring is the practice of continuously observing system resources, processes, and logs to ensure optimal performance, detect problems early, and plan for future capacity needs.

A proactive system administrator monitors systems before users complain.

### Key Reasons

- **Performance Analysis**: Identify CPU, memory, disk, or network bottlenecks.
- **Capacity Planning**: Track trends and plan upgrades.
- **Security Auditing**: Detect suspicious processes, connections, or login failures.
- **Troubleshooting**: Correlate performance issues with resource usage.
- **SLA Compliance**: Verify uptime and performance guarantees.

### The Four Core Metrics

| Metric | What It Measures | Signs of Trouble |
|----------|----------------|----------------|
| CPU | Processor utilization and run queue length | Load average > number of cores |
| Memory | RAM and swap usage | High swap activity |
| Disk I/O | Read/write activity and queue depth | High await, high utilization |
| Network | Throughput and packet loss | Drops, retransmissions |

**Important:** Resource bottlenecks are often related. Low memory can trigger swapping, causing disk I/O delays and increased CPU wait times.

---

## Real-Time Monitoring with top and htop

### The top Command
We talked about this command in processes lecture. Let's quickly go through this:

Launch:

```bash
top
```



#### Line 1

- Uptime
- Number of users
- Load averages (1, 5, 15 minutes)

#### Line 2

Process counts:

- Running
- Sleeping
- Stopped
- Zombie

#### Line 3

CPU States:

| Field | Meaning |
|---------|----------|
| us | User CPU time |
| sy | System CPU time |
| id | Idle |
| wa | I/O Wait |

#### Memory Information

Shows:

- Physical memory
- Swap memory

Focus on **available memory**, not just free memory.

### htop

Install:

```bash
sudo apt install htop
```

or

```bash
sudo yum install htop
```

Launch:

```bash
htop
```

### Advantages

- Colored interface
- Mouse support
- Tree view (F5)
- Easy filtering
- Full command display

---

# 3. Process Snapshot with ps

Unlike `top`, `ps` provides a snapshot of current processes. We talked about this last week but let's briefly look at it again.

```bash
ps aux
```

Options:

- `a` – All users
- `u` – User format
- `x` – Processes without terminals

---

### UNIX Style

```bash
ps -ef
```

Options:

- `-e` – All processes
- `-f` – Full format

## Memory Monitoring

### free Command

### Human Readable

```bash
free -h
```

### Megabytes

```bash
free -m
```

### Refresh Every 5 Seconds

```bash
free -s 5
```

### Sample Output

```text
              total        used        free
Mem:           7.7G        2.1G        1.2G
Swap:          2.0G        0.0B        2.0G
```

### Important Columns

| Column | Meaning |
|---------|----------|
| total | Installed RAM |
| used | Memory in use |
| free | Completely unused |
| buff/cache | Kernel cache |
| available | Memory available without swapping |

---

## vmstat

Example:

```bash
vmstat 2 5
```


## Disk I/O Monitoring
Disk I/O monitoring helps system administrators analyze how efficiently storage devices handle read and write operations. Excessive disk activity, high latency, or I/O bottlenecks can lead to slow application performance and poor system responsiveness. Monitoring disk utilization enables administrators to identify resource-intensive processes, troubleshoot performance issues, and optimize storage usage.


### iostat

Install:

```bash
sudo apt install sysstat
```

or

```bash
sudo yum install sysstat
```

Run:

```bash
iostat -x 1
```

### Important Columns

| Column | Meaning |
|----------|---------|
| r/s | Reads per second |
| w/s | Writes per second |
| rkB/s | Read throughput |
| wkB/s | Write throughput |
| await | Average latency |
| %util | Device utilization |

### Example

```bash
iostat -x 1 5
```

Below is the simplified output of the above command in a clean table format.

| Device | r/s | rkB/s | r_await | w/s | wkB/s | w_await | %util |
|----------|------|--------|----------|------|--------|----------|-------|
| loop0 | 0.00 | 0.00 | 1.07 | 0.00 | 0.00 | 0.00 | 0.00 |
| loop1 | 0.01 | 0.04 | 6.84 | 0.00 | 0.00 | 0.00 | 0.00 |
| loop10 | 0.01 | 0.04 | 8.43 | 0.00 | 0.00 | 0.00 | 0.00 |
| loop11 | 0.08 | 3.03 | 5.57 | 0.00 | 0.00 | 0.00 | 0.03 |


### Capacity Monitoring (df)

df displays the amount of space available on the file system containing each file name argument.  If no file name is given, the space available on all currently mounted file systems is shown.  Space is shown in  1K  blocks  by  default

`-h` parameter print sizes in powers of 1024

```bash
df -h
```

Output:
```text
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           680M  2.7M  678M   1% /run
/dev/sda2        25G  6.5G   17G  28% /
tmpfs           1.7G     0  1.7G   0% /dev/shm
none            1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
tmpfs           1.7G   16K  1.7G   1% /tmp
none            1.0M     0  1.0M   0% /run/credentials/systemd-resolved.service
ITC292          1.9T  563G  1.3T  31% /media/sf_ITC292
tmpfs           340M   92K  340M   1% /run/user/1000
```


### Disk Usage (du)
It’s a command used to show how much disk space files and directories are using.


```bash
du -sh ~/Documents
```

Output:
```text
40K	/home/raziiqbal/Documents

```


## Network and Log Monitoring

### Network Checks

In order to display active network sockets and listening services (basically: what ports your system is using and which programs own them), `ss` command can be used


```bash
ss -tulpn
```

Output (simplified):
```text
Netid    State     Recv-Q    Send-Q                           Local Address:Port          Peer Address:Port    Process                                
udp      UNCONN    0         0                                      0.0.0.0:5353               0.0.0.0:*                                              
udp      UNCONN    0         0                                    10.0.2.15:3702               0.0.0.0:*        users:(("python3",pid=6422,fd=10))    
udp      UNCONN    0         0                              239.255.255.250:3702               0.0.0.0:*        users:(("python3",pid=6422,fd=8))     
udp      UNCONN    0         0                                   127.0.0.54:53                 0.0.0.0:*                                          
```

`ss` stands for socket statistics. In the above command `-t` shows TCP connections, `-u` shows UDP connections, `-l` shows only listening sockets, `-p` shows process using each socket and `-n` shows numeric output.

### Log Monitoring with journalctl
Log monitoring involves collecting, reviewing, and analyzing system and application logs to identify errors. Logs provide valuable insights into system behavior and are essential for troubleshooting and auditing.

`journalctl` is a Linux command used to view system logs collected by the systemd journal. It lets you inspect what your system, services, and applications have been doing (errors, warnings, startup messages, etc.).

```bash
journalctl
```

Output (simplified):
```text
Jun 08 17:42:05 UBuntu26 kernel: Linux version 7.0.0-14-generic (buildd@lcy02-amd64-043) (x86_64-linux-gnu-gcc (Ubuntu 15.2.0-16ubuntu1) 15.2.0, GNU >
Jun 08 17:42:05 UBuntu26 kernel: Command line: BOOT_IMAGE=/boot/vmlinuz-7.0.0-14-generic root=UUID=773874b8-3023-4d5b-bc34-be4f794090a4 ro crashkerne>
Jun 08 17:42:05 UBuntu26 kernel: KERNEL supported cpus:
Jun 08 17:42:05 UBuntu26 kernel:   Intel GenuineIntel
Jun 08 17:42:05 UBuntu26 kernel:   AMD AuthenticAMD
Jun 08 17:42:05 UBuntu26 kernel:   Hygon HygonGenuine
Jun 08 17:42:05 UBuntu26 kernel:   Centaur CentaurHauls

```

Logs can be simplified to show only the last 50 lines

```bash
journalctl -n 50
```

Logs can also be filtered by time.

```bash
journalctl --since "1 hour ago"
```

#### grep
grep is a Linux command used to search text inside files or output. It stands for Global Regular Expression Print. It finds lines that match a pattern. It can be used with other Linux commands as below. For example, in order to show processes related to Firefox only, we can use `grep` with `ps` as below:

```bash
ps aux | grep firefox
```

Output:
```text
raziiqb+    6829 14.5  4.1 2726796 145744 ?      Dl   15:44   0:01 /snap/firefox/8107/usr/lib/firefox/firefox
raziiqb+    6896  0.0  0.0 149388  3056 ?        Sl   15:44   0:00 /snap/firefox/8107/usr/lib/firefox/crashhelper 6829 9 /tmp/ 11
raziiqb+    6925  7.8  1.9 257740 66136 ?        D    15:44   0:00 /snap/firefox/8107/usr/lib/firefox/glxtest -f 25 -w
raziiqb+    6955  0.0  0.0  18000  2676 pts/0    S+   15:44   0:00 grep --color=auto firefox
```

> We'll talk more about grep throughout this course.

## Linux Boot Process Overview

Typically there are six Stages of a boot process in Linux:

1. **BIOS / UEFI**: This is the first firmware step when you power on the computer. It runs immediately after pressing the power button and performs hardware checks (POST): CPU, RAM, disk, etc. Then it finds a bootable device (SSD, HDD, USB) and hands control to the bootloader. 
2. **Bootloader (GRUB2)**: Loads from the disk after BIOS/UEFI and shows a boot menu (if configured). It lets you choose between Linux kernel version, Recovery mode
and other operating systems. Finally, it loads the Linux kernel into memory.
3. **Kernel**: This is the core of the operating system. It takes control of the system, initializes CPU, memory, drivers and mounts the root filesystem (/). Finally, it starts the first user-space process (init or systemd).
4. **init / systemd**: Modern Linux uses systemd (older systems used init). It is the first user-space process (PID 1). It starts and manages all system services like networking, logging, SSH and display manager. It controls system startup in parallel for speed.
5. **Target Activation**: systemd reaches a target state. A 'target' is like a mode of operation: multi-user.target → server mode (no GUI) or graphical.target → desktop GUI mode. It ensures required services are running for that mode.
6. **Login Prompt**: This is the final stage where the system is ready for the user. It authenticates user credentials and starts user session (shell or desktop environment).


## Shutdown and Reboot
It is important to follow the correct procedures when you shut down a Linux system. 
If you fail do so, your filesystems probably will become trashed and the files probably will become scrambled. This is because Linux has a disk cache that won't write things to disk at once, but only at intervals. There can be lots of things going on in the background, and shutting the power can be quite disastrous. By using the proper shutdown sequence, you ensure that all background processes can save their data.


### Traditional shutdown

Shutdown now:

```bash
sudo shutdown -h now
```

Shutdown in 5 minutes:

```bash
sudo shutdown -h +5
```

Reboot now:

```bash
sudo shutdown -r now
```

Cancel:

```bash
sudo shutdown -c
```

### User Login Scripts (`~/.profile`)
When you power on a Linux system, the boot process (BIOS → bootloader → kernel → systemd) ends with a login prompt (text or graphical). After you successfully log in, the system needs to set up your user environment, for example, your command prompt style, search path for programs, and any personal scripts you want to run. This is where login scripts come in. For the Bash shell, one of the most important login scripts is `~/.profile`. It is 
- a plain text file in your home directory.
- executed once every time you log in via a text console or a graphical display manager.
- run before your terminal or desktop starts. 

> Changes take effect the next time you log in. To test immediately, you have to log out and back in.

## Ungraded Exercises
You can perform these exercises on any Linux system (virtual machine, WSL, or a real Linux installation). Root access is not required for most commands, except for installing tools. Use only scripts to complete these exercises.

### Exercise 1: Basic
Check disk usage of any folder using `du` making sure that disk sizes are human readable, e.g., 4.0K etc. Sort it in ascending order, e.g., smallest first.
> Hint: use `| sort -h` to combine the output of the `du` with sort.

### Exercise 2: Intermediate
Use `~/.profile` to append a timestamp to a log file every time you log into the system. This gives you a visible record of your login history, a simple but useful system administration task.

Add a single line to your `~/.profile` that appends a line to a file named `login_history.txt` in your home directory. The line should contain the current date and time in this format:
```text
Login on: YYYY-MM-DD at HH:MM:SS
```
> You can put the entire command directly in `~/.profile`; no separate script file is needed. 

Log out and log back in to verify the file is created and contains the login entry.

### Exercise 3: Challenging
You have a process that consumes more and more memory over time (a “memory hog”). You need to automatically kill it when it uses more than 45000 KB of virtual memory. 
Save the following as `memory_hog.sh` and run it in the background (using `&`). It prints its PID and then slowly consumes memory.

```bash
#!/bin/bash
# memory_hog.sh – consumes ~200 MB then sleeps
array=()
while true; do
    array+=("$(dd if=/dev/zero bs=1M count=200 2>/dev/null | base64 | head -c 1M)")
    sleep 10
done
```
Create a script named `monitor.sh` that:

- Accepts one command‑line argument: the PID of the hog process. Example: `./monitor.sh 12345`
- Checks every 3 seconds how much virtual memory (VSZ) the process is using.
- If the memory usage exceeds 45000 KB
    - Print a message saying the process is being killed.
    - Kill the process using `kill -9`.
    - Exit the script.

- If the process no longer exists (e.g., it died on its own or was killed by someone else), print a message and exit.
- Display a status message every check showing the current memory usage.

Below is the sample output:
```text
raziiqbal@UBuntu26:~/Documents/ITC292/$ ./monitor.sh 16109
PID 16109 uses  41436 KB
PID 16109 uses  42460 KB
PID 16109 uses  42460 KB
PID 16109 uses  42460 KB
PID 16109 uses  43484 KB
PID 16109 uses  43484 KB
PID 16109 uses  43484 KB
PID 16109 uses  43484 KB
PID 16109 uses  44508 KB
PID 16109 uses  44508 KB
PID 16109 uses  44508 KB
PID 16109 uses  45532 KB
Killing 16109:  45532 KB > 45000 KB
[1]+  Killed                     ./memory_hog.sh
```