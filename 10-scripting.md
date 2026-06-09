# 10 · Shell Scripting

> **Module 1 — Putting it all together**
> `intermediate` · ~20 min read

---

## What you'll learn

- How to write a shell script from scratch
- Variables, arguments, and user input
- Conditionals and loops
- Functions
- Real scripts you can actually use

---

## What is a shell script?

A shell script is a text file full of commands — the same commands you'd type in the terminal, but saved so you can run them again (and again, and again) without retyping everything.

If you've ever found yourself running the same sequence of commands more than twice, that's a script waiting to be written.

---

## Your first script

```bash
# Create the file
touch hello.sh

# Open it in a text editor
nano hello.sh
```

Type this:

```bash
#!/bin/bash

echo "Hello, world!"
echo "Today is: $(date)"
echo "You are: $(whoami)"
```

Save it, then make it executable and run it:

```bash
chmod +x hello.sh
./hello.sh
```

```
Hello, world!
Today is: Sun Jun  1 10:00:00 UTC 2025
You are: alice
```

---

## The shebang line

The very first line of every script should be:

```bash
#!/bin/bash
```

This is called the **shebang** (`#!`). It tells the system which interpreter to use to run this file. Without it, the system might try to run your bash script with a different shell and things will break in confusing ways.

```bash
#!/bin/bash        # Use bash (most common)
#!/bin/sh          # Use sh (more portable, fewer features)
#!/usr/bin/env python3  # Use python3 (useful for Python scripts too)
```

---

## Variables

```bash
#!/bin/bash

# Assign a variable (no spaces around the = sign)
name="alice"
age=30
greeting="Hello"

# Use a variable (prefix with $)
echo "$greeting, $name!"
echo "You are $age years old."

# Command substitution — capture output of a command
today=$(date +%Y-%m-%d)
current_user=$(whoami)
file_count=$(ls | wc -l)

echo "Date: $today"
echo "User: $current_user"
echo "Files here: $file_count"
```

> **Always quote your variables** — write `"$name"` not `$name`. If `name` contains spaces, unquoted variables will break your script.

---

## Script arguments

When you run a script, you can pass arguments to it:

```bash
./greet.sh alice 30
#           $1    $2
```

Inside the script:

```bash
#!/bin/bash

# $0 = the script name itself
# $1 = first argument
# $2 = second argument
# $@ = all arguments
# $# = number of arguments

echo "Script name: $0"
echo "First arg:   $1"
echo "Second arg:  $2"
echo "All args:    $@"
echo "Arg count:   $#"
```

### Checking that required arguments were given

```bash
#!/bin/bash

if [ $# -lt 2 ]; then
    echo "Usage: $0 <username> <age>"
    exit 1
fi

name="$1"
age="$2"
echo "Hello, $name! You are $age years old."
```

---

## Conditionals

### `if` statements

```bash
#!/bin/bash

age=25

if [ $age -ge 18 ]; then
    echo "You are an adult."
elif [ $age -ge 13 ]; then
    echo "You are a teenager."
else
    echo "You are a child."
fi
```

### Comparison operators

**Numbers:**

| Operator | Meaning               |
| -------- | --------------------- |
| `-eq`    | equal                 |
| `-ne`    | not equal             |
| `-lt`    | less than             |
| `-le`    | less than or equal    |
| `-gt`    | greater than          |
| `-ge`    | greater than or equal |

**Strings:**

| Operator | Meaning          |
| -------- | ---------------- |
| `=`      | equal            |
| `!=`     | not equal        |
| `-z`     | is empty         |
| `-n`     | is not empty     |

**Files:**

| Operator | Meaning                       |
| -------- | ----------------------------- |
| `-f`     | exists and is a regular file  |
| `-d`     | exists and is a directory     |
| `-e`     | exists (any type)             |
| `-r`     | exists and is readable        |
| `-w`     | exists and is writable        |
| `-x`     | exists and is executable      |

```bash
#!/bin/bash

file="$1"

if [ -z "$file" ]; then
    echo "No file specified."
    exit 1
fi

if [ -f "$file" ]; then
    echo "$file is a regular file."
elif [ -d "$file" ]; then
    echo "$file is a directory."
else
    echo "$file does not exist."
fi
```

---

## Loops

### `for` loop — iterate over a list

```bash
#!/bin/bash

# Loop over a list of values
for colour in red green blue; do
    echo "Colour: $colour"
done

# Loop over files in a directory
for file in /var/log/*.log; do
    echo "Log file: $file"
    wc -l "$file"
done

# Loop using a range
for i in {1..5}; do
    echo "Step $i"
done

# C-style for loop
for ((i=0; i<10; i++)); do
    echo "i = $i"
done
```

### `while` loop — repeat while condition is true

```bash
#!/bin/bash

count=1

while [ $count -le 5 ]; do
    echo "Count: $count"
    count=$((count + 1))
done
```

### `while read` — process a file line by line

```bash
#!/bin/bash

while IFS= read -r line; do
    echo "Line: $line"
done < input.txt
```

This is the correct and safe way to read a file line by line. The `IFS=` prevents leading/trailing whitespace from being stripped, and `-r` prevents backslash interpretation.

---

## Functions

```bash
#!/bin/bash

# Define a function
greet() {
    local name="$1"    # local = only exists inside this function
    echo "Hello, $name!"
}

# Call it
greet "alice"
greet "bob"

# Function with a return value (via stdout)
get_file_size() {
    local file="$1"
    if [ -f "$file" ]; then
        du -sh "$file" | cut -f1
    else
        echo "0"
    fi
}

size=$(get_file_size "/var/log/syslog")
echo "Syslog size: $size"
```

> Use `local` for variables inside functions. Without it, variables are global and can accidentally affect other parts of your script.

---

## Exit codes

Every command returns an exit code. `0` means success. Anything else means failure.

```bash
#!/bin/bash

# Check exit code of last command
ls /nonexistent
echo "Exit code: $?"   # → 2

# Use exit codes in conditionals
if grep -q "error" /var/log/syslog; then
    echo "Errors found in syslog!"
else
    echo "No errors found."
fi

# Exit your script with a specific code
# 0 = success, anything else = failure
exit 0
```

---

## A real script — backup tool

Putting it all together into something you'd actually use:

```bash
#!/bin/bash
# backup.sh — back up a directory with a timestamp

# ── Configuration ──────────────────────────────────────────
SOURCE_DIR="$1"
BACKUP_DIR="${HOME}/backups"
TIMESTAMP=$(date +%Y-%m-%d_%H-%M-%S)
# ───────────────────────────────────────────────────────────

# Validate input
if [ $# -lt 1 ]; then
    echo "Usage: $0 <source-directory>"
    exit 1
fi

if [ ! -d "$SOURCE_DIR" ]; then
    echo "Error: '$SOURCE_DIR' is not a directory."
    exit 1
fi

# Create backup directory if it doesn't exist
mkdir -p "$BACKUP_DIR"

# Get the source directory name (without full path)
DIR_NAME=$(basename "$SOURCE_DIR")
BACKUP_FILE="${BACKUP_DIR}/${DIR_NAME}_${TIMESTAMP}.tar.gz"

# Run the backup
echo "Backing up '$SOURCE_DIR' ..."
tar -czf "$BACKUP_FILE" "$SOURCE_DIR"

# Check if backup succeeded
if [ $? -eq 0 ]; then
    SIZE=$(du -sh "$BACKUP_FILE" | cut -f1)
    echo "Done. Backup saved to: $BACKUP_FILE ($SIZE)"
else
    echo "Error: Backup failed."
    exit 1
fi
```

Run it:

```bash
chmod +x backup.sh
./backup.sh ~/Documents
# → Backing up '/home/alice/Documents' ...
# → Done. Backup saved to: /home/alice/backups/Documents_2025-06-01_10-00-00.tar.gz (4.2M)
```

---

## A real script — system info

```bash
#!/bin/bash
# sysinfo.sh — print a summary of the current system

line() { printf '%*s\n' 40 '' | tr ' ' '─'; }

echo
echo "  System Information"
line

echo "  Hostname   : $(hostname)"
echo "  Kernel     : $(uname -r)"
echo "  Uptime     : $(uptime -p)"
echo "  CPU        : $(grep 'model name' /proc/cpuinfo | head -1 | cut -d: -f2 | xargs)"
echo "  Memory     : $(free -h | awk '/^Mem:/{print $3 " used / " $2 " total"}')"
echo "  Disk (/)   : $(df -h / | awk 'NR==2{print $3 " used / " $2 " total (" $5 " full)"}')"
echo "  Users      : $(who | wc -l) logged in"
echo "  Load       : $(cat /proc/loadavg | cut -d' ' -f1-3)"

line
echo
```

---

## Debugging scripts

```bash
# Run a script in debug mode (prints every command before executing it)
bash -x myscript.sh

# Enable debug mode inside the script
set -x        # turn on
# ... commands ...
set +x        # turn off

# Exit immediately if any command fails (catches errors early)
set -e

# Treat unset variables as an error
set -u

# Best practice — put this at the top of serious scripts
set -euo pipefail
```

---

## Summary

```
Script structure
────────────────────────────────────────
  #!/bin/bash              shebang (always first line)
  set -euo pipefail        exit on errors (recommended)
  name="value"             assign variable
  echo "$name"             use variable
  $(command)               capture command output

Arguments
────────────────────────────────────────
  $0   script name
  $1   first argument
  $#   argument count
  $@   all arguments

Conditionals
────────────────────────────────────────
  if [ condition ]; then
    ...
  elif [ condition ]; then
    ...
  else
    ...
  fi

Loops
────────────────────────────────────────
  for x in a b c; do ... done
  for i in {1..10}; do ... done
  while [ condition ]; do ... done
  while IFS= read -r line; do ... done < file

Functions
────────────────────────────────────────
  fname() { local x="$1"; echo "$x"; }
  result=$(fname "arg")

Exit codes
────────────────────────────────────────
  $?          exit code of last command
  exit 0      success
  exit 1      failure
```

---

## You made it

That's all 10 lessons. Here's where you are now:

```
Chapter 1  ─  Terminal, Filesystem, Files & Dirs    ✓
Chapter 2  ─  Permissions, Users, sudo              ✓
Chapter 3  ─  Pipes, Text Tools, Processes          ✓
Module 1   ─  Shell Scripting                       ✓
```

You now have the foundation to work in the Linux terminal at an intermediate level. You can navigate, manage files, understand who owns what, chain commands into powerful pipelines, process text, control processes, and write scripts that automate real work.

The best way to keep improving is to live in the terminal. Set it as your default way to do things. The discomfort passes quickly.

---

## Where to go next

| Topic              | What to look for                                           |
| ------------------ | ---------------------------------------------------------- |
| Advanced bash      | Arrays, associative arrays, string manipulation, regex     |
| Systemd            | Managing services, timers, writing unit files              |
| Networking         | `ssh`, `curl`, `netstat`, `iptables`, `ss`                 |
| Package management | `apt`, `dnf`, `pacman`, `snap`, `flatpak`                  |
| Text editors       | `vim` or `neovim` — worth the investment                   |
| Version control    | `git` from the terminal (not a GUI)                        |
| Cron jobs          | Scheduling scripts with `crontab -e`                       |

---

*References: [The Linux Command Line by William Shotts](https://linuxcommand.org/tlcl.php) · [GNU Core Utilities](https://www.gnu.org/software/coreutils/) · [Bash Manual](https://www.gnu.org/software/bash/manual/) · [Linux man pages](https://man7.org/linux/man-pages/)*

⬅️ [09 · Processes](../chapter-03/09-processes.md) &nbsp; · &nbsp; [↑ Back to README](../README.md)
