# Week 1 - (Setting up the environment + Working with Terminal)

These notes begin with setup instructions for installing Ubuntu in VirtualBox on Windows, then cover the Linux terminal and the essential commands you will use every day as a system administrator.

## Setting Up Your Environment
Before you can follow along with the hands-on videos, you need a working Ubuntu virtual machine on your Windows computer. This section walks you through the setup in two stages: installing VirtualBox, then installing Ubuntu inside it.

> A virtual machine (VM) is a computer running inside your computer. VirtualBox creates a safe, isolated environment where you can install and experiment with Linux without touching your main Windows system.


### Installing VirtualBox
1.	Go to https://www.virtualbox.org/wiki/Downloads
2.	Under VirtualBox platform packages, click Windows hosts to download the installer.
3.	Run the downloaded `.exe` file and follow the prompts.
4.	Click `Yes` when prompted about network interface reset (this is expected).
5.	Click `Finish` to complete the installation.

### Download the Ubuntu ISO
1.	Go to https://ubuntu.com/download/desktop
2.	Download Ubuntu 26.04 LTS or any latest LTS version based on your hardware architecture.
3.	Save it somewhere easy to find, such as your Downloads folder.

### Create and Configure Virtual Machine
1.	Open VirtualBox and click New
2.	Give your VM a name (e.g., `Ubuntu-SysAdmin`), set Type to `Linux`, OS Distribution to `Ubuntu`, Version to `Ubuntu (64-bit)`.
3.	Select the Ubuntu ISO you downloaded as the ISO Image.
4.	Check `Proceed with Unattended Installation` so you can walk through the setup manually.
5.	Allocate memory: at least `4 GB RAM recommended (4096 MB)`. Assign `2 CPU cores` if your machine allows.
6.	Create a virtual hard disk: `25 GB minimum`, VDI format, dynamically allocated.
7.	Click Finish to create the VM.

### Install Ubuntu
1.	Select your VM in VirtualBox and click Start (if it does not start automatically).
2.	The Ubuntu installer will load.  Select Try or Install Ubuntu.
3.	Choose Install Ubuntu, select your language and keyboard layout.
4.	Choose Erase disk and install Ubuntu (this only erases the virtual disk, not your Windows files).
5.	Set your name, computer name, username, and password.
6.	Wait for installation to complete (~15–30 minutes), then click Restart Now.
7.	When prompted, press Enter to remove the installation medium. Ubuntu will boot to the desktop.

> For screenshots and more detail on each step, refer to the official Ubuntu tutorial: https://documentation.ubuntu.com/desktop/en/latest/tutorial/install-ubuntu-desktop/.

## The Linux Terminal
The terminal (also called the command line or shell) is a text-based interface to your operating system. Instead of clicking icons, you type commands and the system responds with text output. This might feel slower at first, but it becomes dramatically faster and more powerful as you practice and it is the primary tool of every professional system administrator.

### Opening the Terminal in Ubuntu
There are two common ways to open a terminal in your Ubuntu VM:
1.	Keyboard shortcut: `Ctrl + Alt + T`
2.	Right-click on the desktop → Open Terminal

When the terminal opens, you will see a prompt like this:
```bash
student@ubuntu:~$
```
This tells you: your `username (student)`, the hostname of the machine (`ubuntu`), the current directory (`~`  means your home folder), and the prompt symbol (`$` for a normal user, `#` for root).

### Command Structure
Most Linux commands follow this pattern:  
```bash
command  [options]  [arguments]
```
- command: the program to run (e.g., `ls`)   
- options: flags that modify behavior, usually start with `-` or `--` (e.g., `-l`, `--all`)  
- arguments: what to act on (e.g., a file or directory name)

### Example:
```bash
ls  -l  /home/student
```

`ls` is the command, `-l` is an option (long format), and `/home/student` is the argument (the directory to list).

### Core Commands
Below are some of the basic commands used in UNIX based systems:

| Command | Common Usage | Description |
|---|---|---|
| `pwd` | `pwd` | Print the current working directory |
| `cd` | `cd <path>` | Change directory |
| `ls` | `ls [options] [path]` | List directory contents |
| `mkdir` | `mkdir <name>` | Create a new directory |
| `rmdir` | `rmdir <name>` | Remove an empty directory |
| `touch` | `touch <file>` | Create an empty file or update timestamp |
| `cp` | `cp <src> <dest>` | Copy files or directories |
| `mv` | `mv <src> <dest>` | Move or rename files and directories |
| `rm` | `rm [options] <file>` | Remove files or directories |
| `cat` | `cat <file>` | Display file contents |
| `echo` | `echo <text>` | Print text to the screen (or a file) |
| `clear` | `clear` | Clear the terminal screen |

### Commands in Depth

#### pwd: Print Working Directory
Before doing anything else, you need to know where you are. `pwd` prints your current location in the filesystem.
```bash
$ pwd
/home/student
```
The output is an absolute path that starts from the root of the filesystem (`/`).

#### cd: Change Directory
`cd` moves you from one directory to another. It is the most-used navigation command.
```bash
$ cd /etc              # go to /etc (absolute path)
$ cd Documents         # go into Documents (relative path)
$ cd ..                # go up one level
$ cd ../..             # go up two levels
$ cd ~                 # go to your home directory
$ cd -                 # go back to the previous directory
```
- Absolute Path: starts with `/`, always refers to the same location regardless of where you are. Example: `/home/student/Documents`
- Relative Path: starts from your current location. Example: `Documents` (if you are already in `/home/student`)

#### ls: List Directory Content
`ls` shows what is inside a directory. On its own, it lists files and folders in the current directory.
```bash
$ ls                   # lists current directory
$ ls /var/log          # lists a specific directory
$ ls -l                # long format (permissions, size, date)
$ ls -a                # show hidden files (those starting with .)
$ ls -la               # long format + hidden files
$ ls -lh               # long format with human-readable file sizes
```
Sample output of `ls -lh`:
```bash
drwxr-xr-x  2 student student 4.0K May 10 09:00 Documents
-rw-r--r--  1 student student  512 May 10 09:05 notes.txt
```
The first character tells you the type: `d` = directory, `-` = regular file, `l` = symbolic link. The rest are permission bits which we will cover in detail in a later week.

#### mkdir: Make Directory
`mkdir` creates a new, empty directory.

```bash
$ mkdir projects                    # creates one directory
$ mkdir projects/webapp             # creates inside an existing directory
$ mkdir -p projects/webapp/assets   # -p creates parent dirs if needed
```

#### rmdir: Remove Directory
`rmdir` removes an empty directory. If the directory contains any files or subdirectories, it will refuse to remove it.

```bash
$ rmdir old_folder     # removes old_folder only if it is empty
```
To remove a directory and everything inside it, use `rm -r` instead (covered next). Use that option with care.

#### touch: Create a File
`touch` creates an empty file. If the file already exists, it updates its timestamp without changing the content.

```bash
$ touch notes.txt               # creates an empty file called notes.txt
$ touch file1.txt file2.txt     # creates multiple files at once
```

#### cp: Copy
`cp` copies files or directories from one location to another.

```bash
$ cp notes.txt backup.txt              # copy a file
$ cp notes.txt Documents/              # copy into a directory
$ cp -r projects/ projects_backup/     # -r copies a directory recursively
$ cp -i notes.txt backup.txt           # -i prompts before overwriting
```

Key point: the original file is not removed. Both source and destination exist after `cp` runs. If the destination already exists, it will be overwritten silently unless you use `-i`.

#### mv: Move / Rename
`mv` moves a file to a different location, or renames it. Unlike `cp`, the original is removed.

```bash
$ mv notes.txt Documents/          # move file to Documents/
$ mv old_name.txt new_name.txt     # rename a file
$ mv projects/ /var/www/           # move entire directory
$ mv -i source.txt dest.txt        # -i prompts before overwriting
```

#### rm: Remove
`rm` deletes files permanently. There is no Recycle Bin on the Linux command line. Once you run `rm`, the file is gone.

```bash
$ rm notes.txt             # deletes a file
$ rm -i notes.txt          # -i asks for confirmation first
$ rm -r old_projects/      # delete a directory and everything in it
$ rm -ri old_projects/     # recursive + confirm each file
```

Key point: Never run `rm -rf /` or `rm -rf *` without being absolutely certain of what you are deleting. These commands can wipe your entire system or all files in a directory with no recovery. Develop the habit of using `-i` until you are confident.

#### cat: Concatenate / Display
`cat` reads a file and prints its contents to the terminal. The name comes from concatenate as it can also combine multiple files.

```bash
$ cat notes.txt                             # displays file contents
$ cat file1.txt file2.txt                   # displays two files in sequence
$ cat file1.txt file2.txt > combined.txt    # combines into a new file
```

For long files, `cat` will scroll all the way to the bottom. Use `less <file>` instead to scroll through page by page.

#### echo: Print Text
`echo` prints text to the terminal. Simple but very useful, especially in scripts.

```bash
$ echo "Hello, World!"           # prints to screen
$ echo "log entry" > log.txt     # writes text to a file (overwrites)
$ echo "more info" >> log.txt    # appends text to a file
```

The `>` operator redirects output to a file (overwrites). `>>` appends without overwriting. 

#### clear
`clear` clears the terminal screen. The keyboard shortcut `Ctrl + L` does the same thing. Your command history is not deleted. Yyou can still press the `Up Arrow` to recall previous commands.

### Useful Keyboard Shortcuts
Learning these shortcuts will save you enormous amounts of time:

| Shortcut | What it does |
|---|---|
| `↑ / ↓` arrow keys | Scroll through command history |
| `Tab` | Auto-complete a file, directory, or command name |
| `Tab Tab` | Show all possible completions when there are multiple |
| `Ctrl + C` | Cancel the currently running command |
| `Ctrl + L` | Clear the screen (same as `clear`) |
| `Ctrl + A` | Move cursor to start of line |
| `Ctrl + E` | Move cursor to end of line |
| `Ctrl + U` | Delete everything before the cursor |
| `!!` | Re-run the last command |
| `history` | Show list of recent commands |

### Ungraded Exercises
Every week you'll be provided with some ungraded exercises that you can try on your own to get a hold of the concepts taught that week. Please feel free to work on different variations of these exercises.

#### Exercise 1: Basic
Create the following structure inside your home directory:

```bash
~/sysadmin-course/
    week1/
        notes/
        scripts/
```

Use `mkdir -p` to create this entire structure in one command. Verify it was created by navigating into it with `cd` and listing each level with `ls`.  
Hint: use `x/{a, b}` to create two subfolders `a` and `b` inside a folder `x` when using `mkdir -p`.

#### Exercise 2: Intermediate
1.	Navigate to `~/sysadmin-course/week1/notes/`.
2.	Create three empty files: `lecture1.txt`, `lecture2.txt`, `lab1.txt`.
3.	Use `cat` to verify the files exist (they will be empty).
4.	Use `echo` to add some text to `lecture1.txt`:
```bash
$ echo "Week 1 lecture notes" > lecture1.txt
```
5.	View the file content with `cat`.
6.	Append a second line to the same file:
```bash
$ echo "Topic: Linux terminal basics" >> lecture1.txt
```
7.	View the file again. Confirm both lines are present.

#### Exercise 3: Challenging
This is a slightly more open-ended challenge to test your understanding.
1.	Create a new directory called `backup` inside your `week1` folder.
2.	Copy all `.txt` files from the `notes/` directory into `backup/` using a single `cp` command. (Hint: use `cp notes/*.txt backup/`.)
3.	Verify the `backup` directory contains the correct files.
4.	Remove the entire `backup/` directory and its contents using `rm -ri backup/`. The `-i` flag will prompt you for each file..
