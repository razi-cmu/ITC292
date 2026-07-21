# Week 5: Managing User Accounts & Linux Permissions

## Managing User Accounts
When a computer is used by many people it is usually necessary to differentiate between the users, for example, so that their private files can be kept private. This is important even if the computer can only be used by a single person at a time, as with most microcomputers. Thus, each user is given a unique username, and
that name is used to log in. There's more to a user than just a name, however. An account is all the files, resources, and information belonging to one user. 

The Linux kernel itself treats users as mere numbers. Each user is identified by a unique integer, the user id or uid, because numbers are faster and easier for a computer to process than textual names. A separate database outside the kernel assigns a textual name, the username, to each user id. The database contains
additional information as well. To create a user, you need to add information about the user to the user database, and create a home directory for them. 

### `/etc/passwd`

The `/etc/passwd` file stores basic user account information. It is world-readable but no longer contains password hashes instead uses a flag, e.g., `x` to show if the encrypted password is stored in the shadow file. The below command shows the first 5 entries of `passwd` file in the terminal.

```bash
head -5 /etc/passwd
```

Output:
```text
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
```

Information about a specific user can also be fetched from `passwd` file. For example, in order to fetch the information of currently logged in user from this file, we can use `grep` as below:

```bash
grep "$USER" /etc/passwd
```

Output:
```text
raziiqbal:x:1000:1000:raziiqbal:/home/raziiqbal:/bin/bash
```

### `/etc/shadow` (requires sudo)
The `/etc/shadow` file contains encrypted password hashes and password aging information. Only root can read it.

```bash
sudo grep "$USER" /etc/shadow
```
### Group memberships

Groups allow you to grant permissions to multiple users at once. The /etc/group file defines group names, GIDs, and members. The command below shows all the groups the current user is part of.

```bash
cat /etc/group | grep "$USER"
```
Below command can list all the groups:
```bash
groups
```
### User Management Utilities
####  Creating, removing and modifying a user
The `useradd` command adds a new user to the system. Without extra options, it creates minimal entries. The command below creates a user with username `student1`

```bash
sudo useradd student1
```
A password can be set for this new user as below:

```bash
sudo passwd student1
```
A prompt asking for a password and then confirming that password will appear.

On many distributions, `useradd` does not automatically create a home directory or copy skeleton files unless you use the `-m` flag.

```bash
ls -ld /home/student1   # likely does NOT exist!
```

Output:
```text
ls: cannot access '/home/student1': No such file or directory
```

To create a user with a home directory and custom shell `-m` and `-s` parameters are used. Before we do that, let's delete `student1`.

```bash
sudo userdel student1
```

As `student1` user is deleted now, we can create a user with home directory and shell as below:

```bash
sudo useradd -m -s /bin/bash student1
sudo passwd student1
```

We can now verify if the home directory of this new user is created or not.

```bash
sudo ls -ld /home/student1
```

Output:
```text
total 20
drwxr-x--- 2 student1 student1 4096 Jun 12 13:51 .
drwxr-xr-x 4 root     root     4096 Jun 12 13:51 ..
-rw-r--r-- 1 student1 student1  220 Feb 13 07:16 .bash_logout
-rw-r--r-- 1 student1 student1 3771 Feb 13 07:16 .bashrc
-rw-r--r-- 1 student1 student1  807 Feb 13 07:16 .profile
```

`usermod` modifies existing user attributes. The below command changes the shell for the user from `bash` to `sh`.

```bash
sudo usermod -s /bin/sh student1
```

The `-aG` option adds a user to supplementary groups without removing them from existing ones.

```bash
sudo usermod -aG sudo student1
```
> Do not add random users to sudo group. The above is only for demonstration purposes.

### Managing Groups
#### Creating and removing a group
Groups are defined in /etc/group. Use `groupadd` to create a new group.

```bash
sudo groupadd alpha
```

Just like users, groups can be verified as below but from `/etc/group`
```bash
cat /etc/group | grep alpha
```

Output:
```text
alpha:x:1002:
```

`groupdel` removes the group entry. 

```bash
sudo groupdel alpha
```
### Lock and unlock a user

Locking a user prevents login by placing a ! at the beginning of the password hash in `/etc/shadow`.

```bash
sudo passwd -l student1
```

Output:
```text
passwd: password changed.
```
We can check the status of the user as below:

```bash
sudo passwd -S student1
```

Output:
```text
student1 L 2026-06-12 0 99999 7 -1
```

`L` in the above status means Lock.

A locked user can be unlocked as below:

```bash
sudo passwd -u student1
```

Output:
```text
passwd: password changed.
```
We can verify that as below:

```bash
sudo passwd -S student1
```
Output:
```text
student1 P 2026-06-12 0 99999 7 -1
```
`P` means password set (active).


## Interpreting Linux Permissions
Linux determines whether a user or group has access to files, programs, or other resources on a system by checking the overall effective permissions on the resource
The traditional permissions model in Linux is simple. It is based on four access types, or rules. The following access types are possible:
- (r) Read permission: can view content
- (w) Write permission: can modify content, e.g., rename, delete etc.
- (x) Execute permission: can run files or “cd” into directories
- (--) No permission or no access

These permissions are applied to: Owner, Group and Everyone

### Working with Permissions

We will create a temporary workspace for permission experiments.

```bash
cd /tmp
mkdir perm_lab
cd perm_lab

echo "Hello" > file1.txt
echo "Secret" > private.txt
echo "script" > myscript.sh
mkdir mydir
```

`ls -l` shows the permission string, owner, group, size, and modification time.

```bash
ls -l
```

Output:
```
total 12
-rw-rw-r-- 1 raziiqbal raziiqbal  6 Jun 12 14:17 file1.txt
drwxrwxr-x 2 raziiqbal raziiqbal 40 Jun 12 14:17 mydir
-rw-rw-r-- 1 raziiqbal raziiqbal  7 Jun 12 14:17 myscript.sh
-rw-rw-r-- 1 raziiqbal raziiqbal  7 Jun 12 14:17 private.txt
```

Let's review the permission. 
```bash
# Example: 
-rw-r--r-- 1 user group 6 date file1.txt
```
- First character: `-` (file), `d` (directory), `l` (link)
- Next three: owner permissions (`r`, `w`, `x`)
- Next three: group permissions (`r`, `w`, `x`)
- Last three: others permissions (`r`, `w`, `x`)

There are different ways of changing permissions. Let's start with changing permissions symbolically. Symbolic mode uses letters (`u` user, `g` group, `o` others, `a` all) and operators (`+`, `-`, `=`).

The below command will add execute permission for the owner on script.

```bash
chmod u+x myscript.sh
ls -l myscript.sh
```

Output:
```text
-rwxrw-r-- 1 raziiqbal raziiqbal 7 Jun 12 14:17 myscript.sh
```

Let's remove read and write permissions for group and others on `private.txt`

```bash
chmod go-rw private.txt
ls -l private.txt
```

Output:
```text
-rw----r-- 1 raziiqbal raziiqbal 7 Jun 12 14:17 private.txt
```

We can be explicit with permissions by setting exact permissions: `owner=rw`, `group=r`, `others=none` as below:

```bash
chmod u=rw,g=r,o= file1.txt
ls -l file1.txt 
```

Output:
```text
-rw-r----- 1 raziiqbal raziiqbal 6 Jun 12 14:17 file1.txt
```

Permissions work similarly for directories, except the execute permission on a directory is required to `cd` into it or access its contents. Let's see this in action by removing `x` permissions from the user on `mydir` and then trying to enter that directory.

```bash
chmod u-x mydir
ls -ld mydir 
```

Output:
```text
drw-rwxr-x 2 raziiqbal raziiqbal 40 Jun 12 14:17 mydir
```
Entering now in this directory through `cd` would result in permission denied.

```bash
cd mydir
```

Output:
```text
bash: cd: mydir/: Permission denied
```

We can restore these permissions by adding back the `x` permission as below:

```bash

chmod u+x mydir
cd mydir
touch inside.txt
ls -l
```

Output:
```text
total 0
-rw-rw-r-- 1 raziiqbal raziiqbal 0 Jun 12 14:33 inside.txt
```

Numeric (Octal) permission cam also be used instead of symbolic.

```text
r=4, w=2, x=1. 

rwx = 7, r-x = 5, r-- = 4.
````

Let's set some permissions usin these octal digits. Let's give read, write access to the user, read access to the group and read access to others for `file1.txt`.

```bash
# 644 = rw-r--r--
chmod 644 file1.txt
ls -l file1.txt
```

Output:
```text
-rw-r--r-- 1 raziiqbal raziiqbal 6 Jun 12 14:17 file1.txt
```

Let's take another example. Set read, write, execute permissions to user, read and execute permissions to group and others for `myscript.sh`

```bash
# 755 = rwxr-xr-x
chmod 755 myscript.sh
ls -l myscript.sh
```

Output:
```text
-rwxr-xr-x 1 raziiqbal raziiqbal 7 Jun 12 14:17 myscript.sh
```
## Exercise
Write a Bash script that audits user password information on a Linux system.

The script should:
- Display a message indicating that user account checking has started.
- Retrieve all user accounts (exclude system user).
- Only process users who use `/bin/bash` or `/bin/sh` as their login shell.
- For each valid user, check the date when their password was last changed.
- Display the username and the last password change date.
- If a user has never changed their password, display a warning message indicating this.

### Solution
```bash
#!/bin/bash

echo "Checking user accounts..."

getent passwd | awk -F: '$3 >= 1000 && ($7 == "/bin/bash" || $7 == "/bin/sh") {print $1}' | while read -r user
do
	last_change=$(chage -l "$user" | awk -F: '/Last password change/ {print $2}' | xargs)
	if [ "$last_change" = "never" ]; then
		echo "$user has never changed their password"
	else
		echo "$user: $last_change"
	fi
done
```

## Ungraded Exercises
You can perform these exercises on any Linux system (virtual machine, WSL, or a real Linux installation). Root access is not required for most commands, except for installing tools. Use only scripts to complete these exercises.

### Exercise 1: Basic
Create a user with full name, “Steve Jobs” and a home directory `/home/steve`. Make sure to add a password to this user. Create a group called, “Leaders” and make the user “Steve Jobs” part of that group. Feel free to change the shell to `bash`. Login with this new user and do the followings using Linux commands:
- Create 2 files (f1, f2) on the Desktop of this user using touch command
- Change the permission of f1 so that only owner can read, write and execute it.
- Change the permission of f2 so that it can only be read (even by the owner).

> Hint: use `-c` parameter for adding full name of the user in GECOS field.

### Exercise 2: Intermediate
You are the sysadmin for a small team. Two developers (`dev1`, `dev2`) need to collaborate on a shared project folder. A manager (`mgr`) needs read-only access to review their work. No other users should access this folder.

Perform the following steps:  
    1. Create a group named `devteam`.  
    2. Create two users, `dev1` and `dev2`. Both users should have a home directory and `bash` as shell.  
    3. Add `dev1`and `dev2` to `devteam` group.  
    4. Create another user `mgr` with a home directory and `bash` as shell.  
    5. Create a directory structure as `sudo mkdir -p /projects/app`.  
    6. Set the ownership of this directory to `devteam`. You can change the ownership of a file or a directory using `chown`. Review `man chown` to see how to use it to change the ownership of a directory to a group.  
    7. Set permissions to this directory such that ower is `root`, group (`devteam`) has read, write and execute permissions and others have no access to it.  
    8. Apply `setgid` bit which means any file/folder created in this directory will have the same permissions. Use `sudo chmod g+s /projects/app` to set gid bit. Verify the permissions. Should be `drwxrws---`.  
    9. Login as `dev1`. You can use `sudo -u dev1 bash` for a quick test instead of logging in and out.  
    10. Create a file `file1.txt` in this directory. `dev1` should be able to create a file without any problems. Exit as `dev1` once the file is created.  
    11. Login as `mgr`. You can use `sudo -u mgr bash` for a quick test instead of logging in and out. Trying accessing `file1.txt` inside `/projects/app`. Can `mgr` access it? If no, why? Figure out and fix it so that `mgr` can read this file.

### Exercise 3: Challenging
You are the sysadmin for a small finance team. Two clerks (`clerk1`, `clerk2`) need to collaborate on a shared reports folder. A manager (`finance_manager`) needs read‑only access to review their work. No other users should access this folder. Write a bash script that performs all the following steps automatically. Your script must be saved as `setup_finance.sh` and executed with `sudo`.

Your script must perform the following steps:

1. Create a group named `finance_clerks`.
2. Create two users, `clerk1` and `clerk2`. Both users should have a home directory and `bash` as shell.
3. Add `clerk1` and `clerk2` to the `finance_clerks` group.
4. Create another user `finance_manager` with a home directory and `bash` as shell.
5. Create a directory structure: `sudo mkdir -p /finance/reports`.
6. Set the group ownership of this directory to `finance_clerks`. (Use `chgrp` or `chown :group`.)
7. Set permissions on this directory so that the owner is `root`, the group (`finance_clerks`) has read, write and execute permissions, and others have no access (i.e., `770`).
8. Apply the `setgid` bit on `/finance/reports` so that any new file or folder created inside inherits the group. Use `sudo chmod g+s /finance/reports`. Verify the permissions. Should be `drwxrws---`.
9. As user `clerk1` (use `sudo -u clerk1` inside the script):
    - Create a file `report_q1.txt` in `/finance/reports`. 
    - Set the file’s permissions to `664` (rw-rw-r--). 
    - Add a line of text to the file.
10. After the file is created, show the directory listing (`ls -ld /finance/reports`) and the file listing (`ls -l /finance/reports/report_q1.txt`) so the admin can verify.
11. At the end of the script, print a message: "Finance department setup complete. Set passwords manually using: sudo passwd clerk1 clerk2 finance_manager"

## References
- The Linux System Administrator's Guide - (Chapter 11)
- https://www.redhat.com/en/blog/linux-user-group-management
- https://www.geeksforgeeks.org/linux-unix/set-file-permissions-linux/
- https://www.redhat.com/en/blog/linux-file-permissions-explained
- Manual pages: `man passwd`, `man useradd`, `man chmod`, `man shadow`
