# Week 2: Working with Processes and Memory Management

## Understanding Processes

### What is a Process?

A **process** is a program in execution. It is an active entity that includes:

- The program code (executable instructions)
- The current activity (program counter, CPU registers)
- A stack containing temporary data (function parameters, return addresses, local variables)
- A data section (global variables)
- Resources allocated by the kernel (open files, memory segments, etc.)

**Process vs. Program:**
| Program | Process |
|---------|---------|
| Passive | Active |
| Stored on disk | Loaded in memory (RAM) |
| Can be many instances | One instance per execution |
| Example: `/bin/ls` file | Example: the `ls` command when you type it |

### Process Hierarchy and Tree

Linux organizes processes in a **tree structure**:

- Every process has a **parent** (the process that created it).
- A process can have multiple **children**.
- The first process started by the kernel is `systemd` (or `init` on older systems). It has **PID (process id) = 1**.
- All other processes descend from `systemd`.

**Viewing the process tree:**
```bash
pstree
pstree -p   # shows PIDs as well
Example output (simplified):

systemd(1)─┬─systemd-journal(312)
           ├─sshd(800)─┬─sshd(1201)───bash(1202)
           ├─cron(850)
           └─systemd-logind(860)
```
### Process Identifiers (PIDs)
Every process receives a unique Process ID (PID) when it is created.
Finding PIDs:

```bash
ps aux
pidof firefox # shows the pid of firefox if it is running
```
### Process States
A process cycles through several states during its lifetime. You can see the state in the STAT column of `ps`.

| State | Letter | Description |
|-------|--------|-------------|
| Running (or runnable) | R | Actively executing on a CPU or waiting in the run queue |
| Interruptible sleep | S | Waiting for an event (e.g., user input, file I/O). Can be woken by a signal |
| Uninterruptible sleep | D | Waiting directly for hardware (e.g., disk). Cannot be interrupted |
| Stopped | T | Suspended, usually by SIGSTOP or Ctrl+Z |
| Zombie | Z | Process terminated, but entry remains in process table (parent hasn't read exit status) |


## Managing Processes from the Command Line
### Snapshot with `ps`
ps (process status) gives a static snapshot of processes at a given moment. The table below shows some variations of `ps` outputs.

| Command | Description |
|---------|-------------|
| `ps aux` | All processes, user-oriented format (BSD style) |
| `ps -ef` | All processes, full format (Unix style) |
| `ps -e --forest` | Tree view |
| `ps -u username` | Processes owned by a specific user |
| `ps -p PID` | Information about a specific PID |

Running `ps aux` command provides a lot of information about the processes. Table below provides information about each column of the output of this command.

| Column | Description |
|---------|-------------|
| `USER` | Owner of the process |
| `PID` | Process ID |
| `%CPU` | CPU usage since last update |
| `%MEM` | Share of physical memory |
| `VSZ` | Virtual memory size (total, including swapped-out pages) |
| `RSS` | Resident set size (physical RAM currently used) |
| `TTY` | Terminal associated with the process |
| `STAT` | Process state (`R`, `S`, `D`, `T`, `Z`) |
| `START` | Time the process started |
| `TIME` | Cumulative CPU time used |
| `COMMAND` | Command line that started the process |

### Real‑time Monitoring with top and htop
In order to monitor the processes in real-time in Linux, `top` command is used in Terminal.

```bash
top
```
`top` provides interactive commands as well. While `top` command is running, use the keys below to perform operations within `top` command:

| Key | Action |
|-----|--------|
| `M` | Sort by memory usage (descending) |
| `P` | Sort by CPU usage (descending) |
| `k` | Kill a process, will ask for PID and signal number |
| `r` | Renice (change priority) of a process |
| `q` | Quit |


`htop` is an improved version of `top`. You might have to install it before using. Feel free to explore it after installing and running as below.

```bash
sudo apt install htop   # Debian/Ubuntu
htop
```

### Sending Signals with kill, pkill, killall
Signals are software interrupts sent to processes. The kill command is misnamed which means it can send any signal, not just termination. Below is the typical syntax for its usage.


```bash
kill [-signal] PID
pkill [-signal] process_name   # kills all matching processes
killall [-signal] process_name # kills all by exact name
```

## Memory Management in Linux
### Virtual Memory
Linux uses virtual memory to provide each process with its own address space that appears to be contiguous and private. Each process thinks it has the whole address space (e.g., 4GB on 32‑bit systems). However, the processes are protected from one another. Only the parts of memory that are actually used need to be loaded into physical RAM.

### Swapping and Paging
Swap space is a reserved area on disk (a partition or a file) that acts as an extension of RAM. It has several operations

**Page out**: The kernel moves infrequently used memory pages from RAM to swap.

**Page in**: When a process accesses a page that is not in RAM (a page fault), the kernel reads it back from swap.

Swapping and Paging allows the system to run more processes than physical RAM would support. It frees RAM for active processes and disk caching. However, Disk is roughly 1000× slower than RAM. Excessive swapping leads to thrashing which means the system spends more time moving pages than doing useful work.

### Monitoring Memory Usage
Linux provides utilities to monitor memory usage. `free` is one of them that helps in providing the summary of RAM and swap

```bash
free              # in kilobytes
free -h           # human‑readable (K, M, G)
free -h -s 2      # update every 2 seconds
```

Another utility available for monitoring memory is `vmstat`. It provides detailed memory, swap, and CPU statistics.

```bash
vmstat 2 5          # every 2 seconds, 5 times
```

For more detailed info, `meminfo` can be used. You might need to do `sudo` with the command below.

```bash
cat /proc/meminfo 
```

### Process‑level Memory Usage
In `top` or `htop`, the `RES` column shows physical memory used by that process. In `ps aux`, the `RSS` column is the same. The `%MEM` column shows share of total physical RAM. We can find top memory consumers using sorting technique

```bash
ps aux --sort=-%mem | head -10
top -o %MEM          # in top, press M for memory sort
```
In the above command `| head 10` is used to show only 10 processes sorted by memory share.

## Exercise
As a System Administrator, you are required to log certain system activities like the cpu usage, the list of processes utilizing the processor, uptime of the system, the disk usage, and etc. You are required to perform the following activities:
- Create a snapshot of a system using top command. Make sure to run top command in batch mode with only 1 iteration. Save the first 20 lines into a file named cpu_memory.txt
- List the processes in descending order of their CPU usage using ps command. Make sure to list all the processes along with services. Save the first 15 lines into a file named processes.txt
- Save the uptime of the system in uptime.txt. Make sure to only save the uptime in minutes without showing any extra information.
- Identify the devices that have storage more than 20% and save them to disk_usage.txt.

### Solution
```bash
# Save first 20 lines of top output
top -b -n 1 | head -20 > cpu_memory.txt

# Save first 15 processes sorted by CPU usage
ps -eo pid,user,comm,%cpu --sort=-%cpu | head -15 > processes.txt

# Save uptime in minutes only
uptime -p > uptime.txt

df -h | awk '$5+0 >= 20' > disk_usage.txt
```

## Ungraded Exercises

You can perform these exercises on any Linux system (virtual machine, WSL, or a real Linux installation). Root access is not required for most commands, except for installing tools. Use only terminal commands to complete these exercises.

### Exercise 1: Basic

- Open a terminal window and start a process that runs in the background, such as `sleep 1000 &`. `&` helps in running the command in the background.
- Use `ps` to find the `PID` (Process ID) of that sleep command.
- Use the `kill` command to terminate your sleep process.
- Verify using `ps` command that the process has terminated.

### Exercise 2: Intermediate
- Start a simple mathematical loop by running `yes > /dev/null &`. The `yes` command prints 'y' indefinitely by sending it to `/dev/null` in the background e.g., using `&`
- Run this command to see your process and its current "NI" (Nice) value using this command: `ps -o pid,ni,cmd -p [PID_OF_YES]`. Replace `[PID_OF_YES]` with the process ID of `yes`. "NI" represents the priority of the process. Lower the nicer value, higher the priority.
- Now, make the process "nicer" (lower priority) so the rest of your system runs better.
- Run `top` to see if the priority has changed or not.
- Finally, kill the process and verify its termination.

> Note: use `man` command to read the documentation of the commands, e.g., `man ps` shows the documentation of `ps` command. It can be really handy if you dont know how to use a command or what options it offers.


### Exercise 3: Challenging

- Create a directory named `audit_logs` in your current working directory.
- Launch a background process that generated a heavy system load like running `yes` indefinitely.
- Launch another background process that sleeps for 1000 seconds.
- Identify the PIDs of both of these background processes using a process tree.
- Check for the memory space in human readable format and save output of the command in `memory_status.txt` under `audit_logs` using `>`.
- Run `vmstate` 5 times with a 1 second delay and redirect this output to `system_load.txt` under `audit_logs` using `>`.
- Verify that both files now exist in the `audit_logs` by display the content of this directory.
- Check the CPU usage of `yes` process.
- Without killing this process, change the priority of the `yes` process to 10.
- Observe the change, e.g., `top`.
- Kill both the background processes.
- Verify the processes have been killed.

## References
- https://www.geeksforgeeks.org/linux-unix/processes-in-linuxunix/
- https://www.redhat.com/en/blog/linux-command-basics-7-commands-process-management
- https://www.linuxfoundation.org/blog/blog/classic-sysadmin-linux-101-5-commands-for-checking-memory-usage-in-linux
