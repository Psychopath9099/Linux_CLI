# Linux CLI — Quick-Start Cheatsheet

> The **20% of commands** that cover **80% of real terminal work.**
> From first login to intermediate shell use — no fluff.

---

## Table of Contents

- [The Prompt — What You're Looking At](#the-prompt)
- [Navigation](#navigation)
- [Files & Directories](#files--directories)
- [Reading Files](#reading-files)
- [Searching Text — grep](#searching-text--grep)
- [Pipes & Redirection](#pipes--redirection)
- [Permissions](#permissions)
- [Users & sudo](#users--sudo)
- [Processes](#processes)
- [Keyboard Shortcuts](#keyboard-shortcuts)
- [Quick Reference — All Commands](#quick-reference)

---

## The Prompt

When you open a terminal, you will see something like this:

```
alice @ ubuntu : ~ $
  │       │      │  └── $ = regular user (safe)  /  # = root (be careful)
  │       │      └───── ~ = your home directory
  │       └──────────── machine name
  └──────────────────── your username
```

> **Rule:** If the prompt ends with `$` you are a regular user — this is where you should always be. If it ends with `#` you are root. Exit immediately with `exit` until you know exactly what you are doing.

---

## Navigation

```bash
pwd                     # where am I right now?
ls                      # list files in current directory
ls -l                   # long format (permissions, size, date)
ls -la                  # long format + hidden files (dotfiles)
ls -lh                  # long format with human-readable sizes

cd Documents            # go into Documents/
cd ..                   # go up one level (parent directory)
cd ~                    # go home, from anywhere
cd -                    # jump back to previous directory
cd /etc/nginx           # go to an absolute path
```

### Path shortcuts

| Shortcut | Means                                |
| -------- | ------------------------------------ |
| `~`      | Your home directory (`/home/alice`)  |
| `.`      | Current directory                    |
| `..`     | Parent directory (one level up)      |
| `-`      | Previous directory you were in       |
| `/`      | Root — the top of the entire tree    |

### Key directories

| Path         | What lives there                                      |
| ------------ | ----------------------------------------------------- |
| `/home`      | User home directories                                 |
| `/etc`       | System configuration files                            |
| `/var/log`   | Log files — check here when something breaks          |
| `/usr/bin`   | Most user-installed programs                          |
| `/tmp`       | Temporary files — cleared on every reboot             |
| `/dev`       | Hardware as files (your drives, USB, keyboard...)     |

---

## Files & Directories

### Creating

```bash
touch notes.txt              # create an empty file
touch a.txt b.txt c.txt      # create several at once
mkdir projects               # create a directory
mkdir -p a/b/c               # create nested directories in one go
```

### Copying

```bash
cp notes.txt backup.txt      # copy a file
cp notes.txt Documents/      # copy into a directory
cp -r chapter-01/ backup/    # copy an entire directory (-r = recursive)
```

### Moving & Renaming

```bash
mv notes.txt my-notes.txt    # rename a file
mv notes.txt Documents/      # move to a different directory
mv notes.txt Documents/notes-2024.txt   # move AND rename
```

### Deleting

```bash
rm file.txt                  # delete a file (no recycle bin — permanent)
rm file1.txt file2.txt       # delete multiple files
rm -r old-folder/            # delete a directory and everything in it
rmdir empty-folder/          # delete only if the directory is empty
```

> ⚠️ **`rm` is permanent.** There is no undo. Before running `rm -r`, read the path twice.

### Symbolic links

```bash
ln -s /home/alice/projects ~/proj    # create a shortcut (symlink)
```

---

## Reading Files

```bash
cat file.txt                 # print the whole file
cat -n file.txt              # print with line numbers

less file.txt                # interactive reader (q to quit)
                             #   Space = next page
                             #   b     = previous page
                             #   /word = search
                             #   n     = next match

head file.txt                # first 10 lines
head -n 20 file.txt          # first 20 lines
tail file.txt                # last 10 lines
tail -n 50 file.txt          # last 50 lines
tail -f /var/log/syslog      # live stream — new lines appear as written
```

### Finding files

```bash
find . -name "notes.txt"         # find by exact name
find . -iname "notes.txt"        # case-insensitive
find ~ -name "*.txt"             # find all .txt files under home
find /var/log -name "*.log" -mtime -7    # modified in last 7 days
find ~ -size +100M               # files larger than 100 MB
```

---

## Searching Text — grep

```bash
grep "error" file.txt            # lines containing "error"
grep -i "error" file.txt         # case-insensitive
grep -n "error" file.txt         # show line numbers
grep -r "TODO" ~/projects/       # search recursively in a directory
grep -v "DEBUG" app.log          # lines that DON'T match
grep -c "fail" app.log           # count matching lines
grep -l "password" /etc/         # filenames that contain a match

# Extended regex — match multiple patterns
grep -E "error|warning|critical" app.log
```

### Useful grep patterns

```bash
grep "^alice" /etc/passwd        # lines that START with "alice"
grep "bash$" /etc/passwd         # lines that END with "bash"
grep -C 2 "FAILED" deploy.log    # 2 lines of context around each match
```

---

## Pipes & Redirection

The `|` pipe takes the output of one command and feeds it as input to the next.

```
┌─────────┐     ┌─────────┐     ┌─────────┐
│  ls     │ --> │  grep   │ --> │  sort   │
└─────────┘     └─────────┘     └─────────┘
  list files    filter results   sort them
```

### Pipes

```bash
ls /usr/bin | grep "python"          # filter ls output
ps aux | grep nginx                  # find a running process
cat file.txt | sort | uniq           # sort and deduplicate
ls | wc -l                           # count files in directory
```

### Redirection

```bash
ls -la > filelist.txt                # save output to file (overwrites)
date >> activity.log                 # append to file
sort < names.txt                     # use file as input
ls /nonexistent 2> errors.txt        # redirect errors to file
ls /nonexistent 2>/dev/null          # discard errors completely
./build.sh > out.txt 2>&1            # save both output and errors
./build.sh | tee build.log           # show on screen AND save to file
```

### Common pipeline patterns

```bash
# Count something
ls /usr/bin | wc -l

# Find and count unique values
cat access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -10

# Top 5 largest directories
du -sh /var/* 2>/dev/null | sort -rh | head -5

# Find errors in logs
journalctl -b | grep "ERROR" | wc -l
```

---

## Permissions

Run `ls -l` to see permissions:

```
-rwxr-xr--   1   alice   dev   2048   Jun 1   deploy.sh
│ │   │   └── others  (everyone else)
│ │   └──────  group   (the "dev" group)
│ └──────────  owner   (alice)
└────────────  type: - = file  d = directory  l = symlink
```

Each group of 3 characters: `r` = read · `w` = write · `x` = execute · `-` = denied

### Octal values — the ones you'll actually use

| Octal | String       | Use it for                              |
| ----- | ------------ | --------------------------------------- |
| `755` | `rwxr-xr-x`  | Scripts, programs — owner edits, all run |
| `644` | `rw-r--r--`  | Regular files — owner edits, all read   |
| `600` | `rw-------`  | SSH keys, secrets — only you            |
| `700` | `rwx------`  | Private scripts — only you can run      |

> **The maths:** `r=4  w=2  x=1` — add them up per group.
> `rwx = 7` · `rw- = 6` · `r-x = 5` · `r-- = 4` · `--- = 0`

### chmod — changing permissions

```bash
chmod +x script.sh           # make executable (symbolic)
chmod 755 script.sh          # set exact permissions (octal)
chmod 644 file.txt           # readable by all, writable by owner
chmod 600 ~/.ssh/id_rsa      # private — required for SSH keys
chmod -R 755 myproject/      # apply recursively to a directory

chmod u+x file               # add execute for owner
chmod g-w file               # remove write for group
chmod o=r file               # set others to read only
```

### chown — changing ownership

```bash
sudo chown bob file.txt           # change owner
sudo chown bob:developers file    # change owner and group
sudo chown -R www-data /var/www   # recursive
```

---

## Users & sudo

### Who am I?

```bash
whoami                       # your username
id                           # uid, gid, and all groups
groups                       # list groups you belong to
```

### sudo — run commands as root

```bash
sudo apt update              # run one command as root
sudo nano /etc/hosts         # edit a system file
sudo -i                      # open a root shell (exit when done!)
sudo -l                      # list what you are allowed to do
```

> `$` = regular user — safe.
> `#` = root — no restrictions, no undo, no safety net.
> Use `sudo` for the one command that needs it. Never stay in a root shell for routine work.

### User & group management

```bash
sudo useradd -m -s /bin/bash bob     # create a user
sudo passwd bob                      # set their password
sudo usermod -aG sudo alice          # add alice to the sudo group
sudo usermod -aG docker alice        # add alice to the docker group
sudo userdel -r bob                  # delete user and their home directory
```

---

## Processes

### Viewing processes

```bash
ps aux                       # all running processes, full detail
top                          # live view (q to quit, k to kill, M for memory sort)
pgrep nginx                  # find PID by process name
pgrep -l python              # find PID and show process name
ps aux | grep firefox        # manual search
```

### Foreground & background

```bash
./script.sh &                # start directly in background
Ctrl+Z                       # pause current process (sends to background)
bg %1                        # resume job 1 in background
fg %1                        # bring job 1 back to foreground
jobs                         # list all background jobs
nohup ./script.sh &          # keep running after you log out
```

### Stopping processes

```bash
kill 2048                    # politely ask process to stop (SIGTERM)
kill -9 2048                 # force kill — immediate, no cleanup (SIGKILL)
pkill nginx                  # kill by name
pkill -9 python3             # force kill by name
Ctrl+C                       # cancel the current foreground command
```

| Signal    | Number | What it does                                         |
| --------- | ------ | ---------------------------------------------------- |
| `SIGTERM` | 15     | Politely ask to stop — process can clean up (default)|
| `SIGKILL` | 9      | Force stop — cannot be ignored or caught             |
| `SIGHUP`  | 1      | Reload config — used by nginx, sshd, etc.            |

> Use `SIGTERM` first. Only reach for `-9` if it refuses to stop.

---

## Keyboard Shortcuts

> These save more time than any single command. Learn them early.

| Shortcut       | What it does                              |
| -------------- | ----------------------------------------- |
| `Tab`          | Autocomplete command or filename          |
| `Tab Tab`      | Show all possible completions             |
| `↑` / `↓`     | Scroll through command history            |
| `Ctrl + A`     | Jump to start of line                     |
| `Ctrl + E`     | Jump to end of line                       |
| `Ctrl + U`     | Delete everything before the cursor       |
| `Ctrl + K`     | Delete everything after the cursor        |
| `Ctrl + W`     | Delete the word before the cursor         |
| `Ctrl + L`     | Clear the screen                          |
| `Ctrl + C`     | Cancel the current command                |
| `Ctrl + Z`     | Pause current process (then `bg` to resume)|
| `Ctrl + R`     | Search command history interactively      |
| `Ctrl+Shift+C` | Copy in terminal                          |
| `Ctrl+Shift+V` | Paste in terminal                         |

---

## Quick Reference

A single-page summary of everything above.

### Navigation & files

| Command               | Does                                  |
| --------------------- | ------------------------------------- |
| `pwd`                 | Print current directory               |
| `ls -la`              | List all files, long format           |
| `cd ~`                | Go home                               |
| `cd -`                | Go to previous directory              |
| `touch file`          | Create empty file                     |
| `mkdir -p a/b`        | Create nested directories             |
| `cp -r src/ dst/`     | Copy directory recursively            |
| `mv old new`          | Rename or move                        |
| `rm -r folder/`       | Delete directory (permanent)          |
| `find . -name "*.sh"` | Search files by name                  |
| `cat file`            | Print file contents                   |
| `less file`           | Scroll through file interactively     |
| `tail -f logfile`     | Live-stream file updates              |

### Search & text

| Command                    | Does                              |
| -------------------------- | --------------------------------- |
| `grep "pat" file`          | Find lines matching pattern       |
| `grep -r "pat" .`          | Recursive search                  |
| `grep -v "pat" file`       | Lines that don't match            |
| `sort file`                | Sort lines alphabetically         |
| `sort -rn file`            | Sort numerically, reversed        |
| `sort \| uniq -c`          | Count duplicates                  |
| `wc -l file`               | Count lines                       |
| `cut -d: -f1 file`         | Extract first colon-delimited col |
| `awk '{print $1}' file`    | Print first field of each line    |
| `sed 's/old/new/g' file`   | Find and replace                  |

### Permissions

| Command                 | Does                              |
| ----------------------- | --------------------------------- |
| `chmod +x file`         | Make executable                   |
| `chmod 755 file`        | `rwxr-xr-x`                       |
| `chmod 644 file`        | `rw-r--r--`                       |
| `chmod 600 file`        | `rw-------` (private)             |
| `chown user:grp file`   | Change owner and group            |

### Processes & system

| Command              | Does                                 |
| -------------------- | ------------------------------------ |
| `ps aux`             | All running processes                |
| `top`                | Live process viewer                  |
| `pgrep name`         | Find PID by name                     |
| `kill PID`           | Politely stop process                |
| `kill -9 PID`        | Force stop process                   |
| `cmd &`              | Run in background                    |
| `jobs`               | List background jobs                 |
| `nohup cmd &`        | Survive terminal close               |
| `whoami`             | Current username                     |
| `sudo command`       | Run as root                          |
| `df -h`              | Disk space usage                     |
| `free -h`            | Memory usage                         |
| `uptime`             | How long the system has been running |

---

## Things to remember

```
The terminal does exactly what you tell it.
It will not ask "are you sure?" for most things.
rm has no recycle bin.
# means root means no limits — exit quickly.
Tab autocompletes everything — use it always.
↑ scrolls history — you rarely need to retype a command.
man <command> opens the full manual for any command.
```

---

*Part of [LinuxLore](./README.md) · Based on the spirit of [The Linux Command Line](https://linuxcommand.org/tlcl.php) by William Shotts · GNU/Linux · Open Source*
