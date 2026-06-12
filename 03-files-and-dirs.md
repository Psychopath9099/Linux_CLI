# 03 · Files & Directories

> **Chapter 1 of 3 — The Shell & Filesystem**  
> `beginner` · ~12 min read

---

## What you'll learn

- Create, copy, move, rename, and delete files and directories
    
- Read file contents in different ways
    
- Find files quickly with `find`
    
- Understand when to be careful (and why `rm` has no recycle bin)
    

---

## Creating things

### `touch` — create an empty file

```bash
# Create a single file
touch notes.txt

# Create several at once
touch file1.txt file2.txt file3.txt
```

> `touch` was originally designed to update a file's timestamp. Creating a new file is just a side effect — if the file doesn't exist yet, it gets created.

Files whose names begin with a dot (`.`) are hidden by default.

```bash
touch .secret
```

---

### `mkdir` — create a directory

```bash
# Create a single directory
mkdir projects

# Create nested directories in one go (-p = parents)
mkdir -p projects/linux-lore/notes

# Create several sibling directories
mkdir chapter-01 chapter-02 chapter-03
```

> **Remember:** `touch` creates files. `mkdir` creates directories.

Without `-p`, this would fail if `projects` doesn't already exist:

```bash
# This FAILS if projects/ doesn't exist
mkdir projects/linux-lore

# This WORKS because -p creates the whole path
mkdir -p projects/linux-lore
```

---

## Copying things

### `cp` — copy a file or directory

```bash
# Copy a file
cp notes.txt notes-backup.txt

# Copy a file into a directory
cp notes.txt backups/

# Copy and rename at the same time
cp notes.txt backups/notes-june.txt

# Copy an entire directory (-r = recursive)
cp -r chapter-01/ chapter-01-backup/
```

> Always use `-r` when copying directories. Without it, `cp` refuses to copy a folder.

---

## Moving and renaming

### `mv` — move or rename

`mv` does two jobs. Same command, different use:

```bash
# Rename a file
mv notes.txt my-notes.txt

# Move a file to a different directory
mv notes.txt Documents/

# Move AND rename at the same time
mv notes.txt Documents/notes-2024.txt

# Move a whole directory
mv chapter-01/ archive/
```

```text
Before:  ~/notes.txt
After:   ~/Documents/notes-2024.txt

mv notes.txt Documents/notes-2024.txt
```

---

## Deleting things

### `rm` — remove files and directories

> **Warning:** `rm` is permanent. There is usually no recycle bin in the terminal. Once you run `rm`, the file is gone.

```bash
# Delete a single file
rm notes.txt

# Delete multiple files
rm file1.txt file2.txt file3.txt

# Delete a directory and everything in it (-r = recursive)
rm -r old-projects/

# Ask for confirmation before each deletion (-i = interactive)
rm -i important.txt
```

```bash
# The dangerous pattern — never run this without thinking twice:
rm -rf /some/directory

# -r = recursive (delete everything inside)
# -f = force (no prompts, no errors)
# Together: silently delete everything, no questions asked
```

> **Rule of thumb:** If you're about to run `rm -rf`, pause and read the path twice. There's no undo.

---

### `rmdir` — remove an empty directory

```bash
# Only works if the directory is completely empty
rmdir empty-folder/
```

Use `rm -r` if you want to delete a directory that still has files in it.

---

## Reading files

### `cat` — print a file's contents

```bash
# Print a file
cat notes.txt

# Print with line numbers
cat -n notes.txt

# Print two files one after another
cat file1.txt file2.txt
```

`cat` is short for _concatenate_ — its real purpose is to join files together. But printing a single file is what most people use it for day-to-day.

---

### `less` — read long files comfortably

```bash
less ~/.bashrc
```

```bash
less /etc/hosts
```

`less` is an interactive file viewer. Useful when a file is too long to fit on screen.

|Key|Action|
|---|---|
|`Space`|Next page|
|`b`|Previous page|
|`↑` `↓`|Scroll one line at a time|
|`/word`|Search for "word"|
|`n`|Jump to next search match|
|`q`|Quit|

---

### `head` and `tail` — see just the top or bottom

```bash
# First 10 lines of a file (default)
head notes.txt

# First 20 lines
head -n 20 notes.txt

# Last 10 lines
tail notes.txt

# Last 50 lines
tail -n 50 notes.txt

# Watch a file live as it gets updated
tail -f example.log
```

`tail -f` is one of the most useful commands when debugging — it streams new lines as they get added to a file in real time.

Press `Ctrl + C` to stop following the file.

> **Note:** Depending on your Linux distribution, log files may have different names and locations.

---

## Finding files

### `find` — search the filesystem

```bash
# Find a file by name in the current directory (and subdirectories)
find . -name "notes.txt"

# Case-insensitive search
find . -iname "notes.txt"

# Find all .txt files under your home directory
find ~ -name "*.txt"

# Find files modified in the last 7 days
find /var/log -name "*.log" -mtime -7

# Find files larger than 100MB
find ~ -size +100M

# Find and list with details
find . -name "*.sh" -ls
```

The `*` is a **wildcard** — it matches anything. `*.txt` means "any file that ends in .txt".

---

## Symbolic links

A symbolic link (symlink) is like a shortcut — a pointer to another file or directory.

```bash
# Create a symlink
ln -s ~/projects/linux-lore ~/lore

# Now ~/lore points to the full path
cd ~/lore
```

You'll see symlinks a lot in `/usr/bin` — many commands are symlinks that point to the actual binary elsewhere.

---

## Practical example — a real session

```bash
# Start a new project from scratch
shadows@ubuntu:~$ mkdir -p projects/website/assets
shadows@ubuntu:~$ cd projects/website

# Create some files
shadows@ubuntu:~/projects/website$ touch index.html style.css
shadows@ubuntu:~/projects/website$ ls
assets/  index.html  style.css

# Put a copy of style.css in assets too
shadows@ubuntu:~/projects/website$ cp style.css assets/style-backup.css

# Rename index.html
shadows@ubuntu:~/projects/website$ mv index.html home.html

# Check what we have
shadows@ubuntu:~/projects/website$ ls -la
total 16
drwxr-xr-x 3 shadows shadows 4096 Jun  1 10:01 .
drwxr-xr-x 3 shadows shadows 4096 Jun  1 10:00 ..
drwxr-xr-x 2 shadows shadows 4096 Jun  1 10:01 assets
-rw-r--r-- 1 shadows shadows    0 Jun  1 10:00 home.html
-rw-r--r-- 1 shadows shadows    0 Jun  1 10:00 style.css
```

---

## Summary

```text
Creating
────────────────────────────────────────
  touch file.txt        create empty file
  mkdir -p a/b/c        create directories (nested)

Copying & Moving
────────────────────────────────────────
  cp file.txt copy.txt  copy a file
  cp -r dir/ backup/    copy a directory
  mv old.txt new.txt    rename
  mv file.txt dir/      move to directory

Deleting
────────────────────────────────────────
  rm file.txt           delete a file (permanent!)
  rm -r folder/         delete a directory
  rmdir folder/         delete ONLY if empty

Reading
────────────────────────────────────────
  cat file.txt          print whole file
  less file.txt         interactive reader
  head -n 20 file.txt   first 20 lines
  tail -f logfile       live stream updates

Finding
────────────────────────────────────────
  find . -name "*.txt"  search by name
  find ~ -mtime -7      modified in last 7 days
  find ~ -size +100M    larger than 100 MB
```

---

## Chapter 1 Complete — The Shell & Filesystem

You can now navigate the filesystem, read and create files, and move things around. That's the foundation. Chapter 2 goes deeper — who owns those files, and who is allowed to access or modify them.

➡️ [Chapter 2 → 04 · Permissions](04-permissions.md)

⬅️ [02 · The Filesystem](02-filesystem.md)