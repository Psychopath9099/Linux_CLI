# 02 · The Filesystem

> **Chapter 1 of 3 — The Shell & Filesystem**
> `beginner` · ~12 min read

---

## What you'll learn

- How the Linux directory tree is structured
- What the important directories actually contain
- Absolute vs relative paths
- How to navigate confidently with `pwd`, `ls`, `cd`

---

## Everything is a file

This is the most important thing to understand about Linux:

> **Everything is a file.** Your documents, yes. But also your programs, your settings, your USB drive, your keyboard, your network card — all of them appear somewhere in the filesystem as files.
> Technically, not everything is a regular file, but Linux exposes many system resources through file-like interfaces.

This sounds weird at first. But it's what makes Linux so powerful and composable. If you can read a file, you can read from hardware. If you can write to a file, you can configure the system.

---

## The directory tree

Unlike Windows (where each drive gets a letter like `C:\` or `D:\`), Linux has a single tree that starts at one point: **`/`**, called the **root**.

Everything branches from there:

```
/
├── etc/          ← system configuration files
├── home/         ← user home directories
│   ├── shadows/
│   └── bob/
├── var/
│   └── log/      ← system logs
├── usr/
│   ├── bin/      ← user programs (ls, grep, python3...)
│   └── lib/      ← shared libraries
├── bin/          ← essential system programs
├── tmp/          ← temporary files
├── dev/          ← device files (your hardware)
├── proc/         ← virtual filesystem, live kernel info
└── root/         ← home directory of the root user
```

---

## Key directories you need to know

| Path       | What lives there                                                 |
| ---------- | ---------------------------------------------------------------- |
| `/`        | Root — the top of everything                                     |
| `/home`    | Your stuff. shadows's files live in `/home/shadows`                  |
| `/etc`     | System config files. nginx config, fstab, hostname — all in here |
| `/bin`     | common command-line programs                                     |
| `/usr/bin` | most command-line programs                                       |
| `/var/log` | Log files. When something breaks, look here first                |
| `/tmp`     | Temporary files. Often cleared automatically                     |
| `/dev`     | Hardware as files — `/dev/sda` is your first hard drive          |
| `/proc`    | Live kernel data — `/proc/cpuinfo` shows your CPU in real time   |
| `/root`    | The root user's home directory (not the same as `/`)             |

> **Common mistake:** `/root` and `/` are different things. `/` is the top of the entire tree. `/root` is just a home folder for the superuser.

---

## Navigating the tree

### `pwd` — where am I?

```bash
pwd
```

```
/home/shadows
```

`pwd` stands for **Print Working Directory**. It tells you exactly where you are in the tree right now.

---

### `ls` — what's in here?

```bash
# Basic list
ls

# Long format (shows permissions, size, date)
ls -l

# Long format + hidden files (files starting with .)
ls -la

# Human-readable file sizes
ls -lh
```

Example output of `ls -la`:

```
total 48
drwxr-xr-x  6 shadows shadows 4096 Jun  1 09:12 .
drwxr-xr-x  4 root  root  4096 May 30 18:00 ..
-rw-r--r--  1 shadows shadows  220 May 30 18:00 .bash_logout
-rw-r--r--  1 shadows shadows 3526 May 30 18:00 .bashrc
drwxr-xr-x  2 shadows shadows 4096 Jun  1 08:00 Documents
drwxr-xr-x  2 shadows shadows 4096 Jun  1 08:00 Downloads
```

Don't worry about all of that yet — permissions get their own full lesson.

The columns read: `permissions · links · owner · group · size · date · name`

> [!Note] 
> Files beginning with a dot (`.`) are hidden by default.


---

### `cd` — move around

```bash
# Go into a directory
cd Documents

# Go back to home, no matter where you are
cd ~
# or just
cd

# Go up one level (to the parent directory)
cd ..

# Go back to where you just were
cd -

# Go to an absolute path
cd /etc
```

---

## Absolute vs relative paths

This is a concept that trips up a lot of beginners. Once it clicks, navigation becomes effortless.

**Absolute path** — starts from root `/`. Always works, regardless of where you are.

```bash
cd /home/shadows/Documents
ls /var/log
cat /etc/hostname
```

**Relative path** — starts from where you currently are.

```bash
# If you're in /home/shadows, these two are the same:
cd Documents
cd /home/shadows/Documents
```

### Special path shortcuts

| Shortcut | Means                                |
| -------- | ------------------------------------ |
| `~`      | Your home directory (`/home/shadows`)  |
| `.`      | The current directory                |
| `..`     | The parent directory (one level up)  |
| `-`      | The previous directory you were in   |

```bash
# Practical examples
cd ~/Downloads         # go to your Downloads folder
ls ../                 # list the parent directory
cp file.txt ./backup/  # copy to a subfolder here
cd -                   # jump back to last location
```

---

## Putting it together

Here's a typical navigation session:

```bash
# Start — where am I?
shadows@ubuntu:~$ pwd
/home/shadows

# Look around
shadows@ubuntu:~$ ls
Documents  Downloads  Music  Pictures  Desktop

# Go into Documents
shadows@ubuntu:~$ cd Documents
shadows@ubuntu:~/Documents$ ls
notes.txt  projects/

# Go into projects, then back up two levels at once
shadows@ubuntu:~/Documents$ cd projects
shadows@ubuntu:~/Documents/projects$ cd ../..

# Now I'm back at home
shadows@ubuntu:~$ pwd
/home/shadows

# Jump to a system directory and look around
shadows@ubuntu:~$ ls /etc
hostname  hosts  passwd  shadow  fstab  ...

# Return home instantly
shadows@ubuntu:~$ cd ~
```

---

## Summary

```
Key commands
────────────────────────────────────────
  pwd          print current directory
  ls -la       list all files, long format
  ls -lh human-readable sizes
  cd <dir>     move into a directory
  cd ~         go home
  cd ..        go up one level
  cd -         go to previous directory

Path types
────────────────────────────────────────
  /home/shadows  absolute (starts from /)
  Documents    relative (from current dir)
  ~            shortcut for your home
  ..           shortcut for parent dir
```

---
### Try it yourself

```bash
pwd
ls
cd /tmp
pwd
cd ~
pwd
```

---

## What's next

Now that you can navigate, you need to actually do things — create, copy, move, and delete files. That's next.

➡️ [03 · Files & Directories](./03-files-and-dirs.md)

⬅️ [01 · The Terminal](./01-the-terminal.md)
