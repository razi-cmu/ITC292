# Week 3: Basics of Shell Scripting

## Introduction to Shell Scripting

### What is a Shell Script?

A shell script is a text file containing a series of commands that the shell executes sequentially. It allows you to automate tasks that you would otherwise type manually at the command prompt.

### Choosing a Shell

- **bash** (Bourne Again Shell): most common, default on Linux.
- **sh** (Bourne Shell): portable, but fewer features.

We will use `bash` for this course.

### Your First Script

Create a file named `hello.sh`:

```bash
#!/bin/bash
# This is a comment
echo "Hello, system administrator!"
```

Notes:

- The shebang (`#!`) must be the first line.
- Comments start with `#` (except the shebang).

Make it executable:

```bash
chmod +x hello.sh
```

Run it:

```bash
./hello.sh

# or

bash hello.sh
```

Output:
```text
Hello, system administrator!
```

## Variables and Data

### Defining Variables
A shell variable is a named container used to store data in memory while a shell script is running. Variables can store string, number, filename or even the output of a command. The equals sign (=) is used with no spaces on either side to declare a variable.

```bash
name="Alice"
age=25

# No spaces around =
```

Use the variable with `$`:

```bash
echo "User: $name, age: $age"
```

Output:
```text
Alice, 25
```

Variable names can contain letters (a-z, A-Z), numbers (0-9) and underscores (_). The names of the variables must start with a letter or underscore. Special characters should be avoided. Shell variable names are case-sensitive which means `var` and `Var` are two different variables.


### Special Variables (Positional Parameters)

When you run a script with arguments, they are available as `$1`, `$2`, ...

Example script `greet.sh`:

```bash
#!/bin/bash
echo "Hello, $1! You are $2 years old."
```

Run:

```bash
./greet.sh Alice 25
```

Output:

```text
Hello, Alice! You are 25 years old.
```

| Variable | Meaning |
|----------|----------|
| `$0` | Name of the script |
| `$1...$9` | First nine arguments |
| `$#` | Number of arguments |
| `$@` | All arguments as separate strings |
| `$?` | Exit status of last command |
| `$$` | Process ID of the script |

### Reading User Input

```bash
read -p "Enter your name: " name
echo "Your name is: $name"
```

### Conditional Statements
Conditional statements, or if-then blocks, are the most important part of any script.
They are the "decision-makers" that allow your script to run, skip, or change code based on a specific condition.

###  if Statement Structure

```bash
if [ condition ]; then
    commands
elif [ other_condition ]; then
    commands
else
    commands
fi
```

**Important:** Spaces inside `[ ]` are mandatory.

### String Comparisons

| Operator | Meaning |
|-----------|----------|
| `"$a" = "$b"` | Strings equal |
| `"$a" != "$b"` | Not equal |
| `-z "$a"` | String is empty |
| `-n "$a"` | String is not empty |

Example:

```bash
if [ "$name" = "root" ]; then
    echo "Welcome, administrator"
fi
```

### Numeric Comparisons

| Operator | Meaning |
|-----------|----------|
| `-eq` | Equal |
| `-ne` | Not equal |
| `-lt` | Less than |
| `-le` | Less or equal |
| `-gt` | Greater than |
| `-ge` | Greater or equal |

Example:

```bash
#!/bin/bash
read -p "Enter your name: " name
read -p "Enter your age: " age

if [ $age -ge 18 ]; then
	echo "You are eligible"
else
	echo "You are not eligible"
fi
```

Output:
```text
Enter your name: razi
Enter your age: 20
You are eligible
```

### Combining Conditions

#### Logical AND (&&):
Runs the second command only if the first command succeeds (exit status 0).
Useful for chaining commands that must all succeed.

```bash
#!/bin/bash
read -p "Enter your name: " name
read -p "Enter your age: " age

if [[ "$name" = "admin" && $age -ge 18 ]]; then
	echo "You are eligible"
else
	echo "You are not eligible"
fi
```

Output:
```text
Enter your name: admin 
Enter your age: 20
You are eligible
```

**Important**: Note those extra square brackets (`[]`) inside the if square brackets.

#### Logical OR (||):
Runs the second command only if the first command fails (non-zero exit status).
Useful for fallback or error handling.

```bash
#!/bin/bash
read -p "Enter your name: " name
read -p "Enter your age: " age

if [[ "$name" = "admin" || "$name" = "razi" ]]; then
	echo "You are eligible"
else
	echo "You are not eligible"
fi
```

Output:
```text
Enter your name: razi
Enter your age: 20
You are eligible
```

You can chain many if elif else statements as per your need.

### File and Directory Tests

| Test | Meaning |
|--------|----------|
| `-e file` | File exists |
| `-f file` | File exists and is regular |
| `-d dir` | Directory exists |
| `-r file` | Readable |
| `-w file` | Writable |
| `-x file` | Executable |

Example:

```bash
#!/bin/bash

if [ -e "hello.sh" ]; then
	echo "File exists"
else
	echo "No such file exists"
fi
```

Output:
```text
File exists
```

## Loops
Loops are a fundamental part of programming, and shell scripting is no exception. 
Loops allow you to automate repetitive tasks by running a block of code multiple times.

### for Loop
Iterate over a list of items:

```bash
for item in apple banana cherry; do
    echo "Fruit: $item"
done
```

Output:
```text
Fruit: apple
Fruit: banana
Fruit: orange
```

for Loop is very handy for iterating over files:

```bash
for file in /var/log/*.log; do
    echo "Processing $file"
done
```

Output:
```text
Processing /var/log/alternatives.log
Processing /var/log/apport.log
Processing /var/log/auth.log
Processing /var/log/bootstrap.log
Processing /var/log/cloud-init.log
Processing /var/log/cloud-init-output.log
Processing /var/log/dpkg.log
Processing /var/log/fontconfig.log
Processing /var/log/gpu-manager.log
Processing /var/log/kern.log
Processing /var/log/vboxadd-install.log
Processing /var/log/vboxadd-setup.log
Processing /var/log/vboxpostinstall.log
```

Scripting allows C-style for loop as well:

```bash
for ((i=1; i<=5; i++)); do
    echo "Number $i"
done
```

Output:
```text
Number 1
Number 2
Number 3
Number 4
```

### while Loop
while loop helps in running code as long as a condition is true.

```bash
counter=1

while [ $counter -le 5 ]; do
    echo "Counter: $counter"
    ((counter++))
done
```

Output:
```text
Counter: 1
Counter: 2
Counter: 3
Counter: 4
Counter: 5
```

while loop is very handy when it comes to reading files line by line:

```bash
#!/bin/bash

counter=1
while read line; do
	echo "Line $counter: $line"
	((counter++))
done < hello.sh
```

Output:
```text
Line 1: #!/bin/bash
Line 2: 
Line 3: counter=1
Line 4: while read line; do
Line 5: echo "Line $counter: $line"
Line 6: ((counter++))
Line 7: done < hello.sh
```

### until Loop
until loop runs code until a condition becomes true.

```bash
#!/bin/bash

counter=1
until [ $counter -ge 5 ]; do
    echo "Still less than 5: $counter"
    ((counter++))
done
```

Output:
```text
Still less than 5: 1
Still less than 5: 2
Still less than 5: 3
Still less than 5: 4
```

## Exit Status and Error Handling

### Exit Codes

- `0` – success
- `1–255` – failure

Check exit code:

```bash
#!/bin/bash

ls ~/Documents
echo "Exit code: $?"
```

Output:
```text
ITC292
Exit code: 0
```

## Functions
Functions are the building blocks of efficient shell scripting. A function is a reusable block of code that performs a specific task. Functions provide following benefits
- Reusability: Write a piece of code once and call it multiple times.
- Readability: Break a long, complex script into logical, named chunks
- Modularity: Easily manage and debug your code by isolating tasks.

The function name can be any valid string and the body of the function can include any sequence of valid shell commands.

Below is the structure of a typical function:

```bash
function_name() {
    # commands
    return value
}
```

Below is a simple example of a function:

```bash
#!/bin/bash

greet(){
	echo "Hello"
}
greet
```

Output:
```text
Hello
```

We can pass arguments to a function as well:

```bash
#!/bin/bash

check_file(){
	if [ -e "$1" ]; then
		echo "File Exists"
	else
		echo "No such file exists"
	fi
}

check_file "hello.sh"
```

Output:
```text
File Exists
```

Functions can be directly called inside a conditional statement.

```bash
#!/bin/bash

is_root(){
	[ $EUID -eq 0 ]
}

if is_root; then
	echo "running as root"
else
	echo "not running as root"
fi
```

Output:
```text
not running as root
```
## Exercise
Write a Bash script named process_monitor.sh that:
- Displays a title "Process Monitor" and a formatted table header.
- Uses the `ps` command to list all running processes.
- Uses `awk` to display only processes that:
	- are not owned by root,
	- have CPU usage greater than 0%, and
	- have memory usage greater than 0%.
- Formats the output into aligned columns showing:
	- Process ID (PID)
	- User
	- CPU Usage
	- Memory Usage
- At the end of the output, displays the total number of processes that matched the criteria.

## Ungraded Exercises
You can perform these exercises on any Linux system (virtual machine, WSL, or a real Linux installation). Root access is not required for most commands, except for installing tools. Use only scripts to complete these exercises.

### Exercise 1: Basic
Create a script `greet.sh` that:

- Accepts a name as an argument
- Prints:

```text
Hello, <name>! Today is <date>.
```

- Uses `date`
- Prints a different message if no argument is given

> Hint: use `$(date)` to get today's date.

### Exercise 2: Intermediate
Create `backup.sh` that:

- Takes a source directory as the first argument
- Takes a destination directory as the second argument
- Checks if the source directory exists
- Creates a copy-based backup folder in the destination using a timestamp format:

```text
backup_YYYY-MM-DD_HH-MM-SS
```

- Uses `cp -r` to copy the entire directory into the backup folder
- Prints a success message if the backup is created
- Prints an error message if the source directory does not exist

> Hint: Use this to format the date in YYYY-MM-dd_HH-MM-S format: `$(date "+%Y-%m-%d_%H-%M-%S")`

### Exercise 3: Challenging

Create a Bash script that:

- Displays a menu with the following options:
```text
1. Show Disk Usage
2. Show Memory Usage
3. Show System Uptime
4. Exit
```
- Repeats the menu continuously using a loop (`while true`)
- Uses read to get user input
- Uses an if / elif / else structure to handle the user’s choice:
```text
1 → display disk usage using (df -h)
2 → display memory usage using (free -h)
3 → display system uptime using (uptime)
4 → exit the script
any other input → print an error message like "Wrong option selected!"
```
The menu should keep reappearing after each action until the user chooses Exit (4)


# Additional Resources

- [Bash Guide for Beginners (TLDP)](https://tldp.org/LDP/Bash-Beginners-Guide/html/)
- [Bash Scripting Fundamentals](https://www.geeksforgeeks.org/linux-unix/bash-scripting-introduction-to-bash-and-bash-scripting/)
- [Bash Tutorial](https://www.w3schools.com/bash/)
