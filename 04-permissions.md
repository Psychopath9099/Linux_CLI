# 04 · Permissions

> **Chapter 2 of 3 — Permissions & Users**
> `beginner → intermediate` · ~13 min read

---

## What you'll learn

- How to read the permission string (`rwxr-xr-x`)
- The difference between owner, group, and others
- How to change permissions with `chmod` (symbolic and octal)
- Why permissions matter in practice

---

## Why permissions exist

Linux is a **multi-user system**. Multiple people (or programs) can use the same machine at the same time. Permissions are how the system decides:

- Who can **read** a file
- Who can **write** to (modify) a file
- Who can **execute** a file as a program

Without permissions, any user could read your private files, overwrite system programs, or execute code they shouldn't touch.

---

## Reading permission strings

Run `ls -l` on any directory:

```
-rwxr-xr--  1  shadows  dev  2048  Jun 1 10:00  deploy.sh
drwxr-xr-x  2  shadows  dev  4096  Jun 1 09:00  scripts/
lrwxrwxrwx  1  shadows  dev    14  Jun 1 08:00  latest -> scripts/v2
```

The first column is the permission string. Let's decode it:

```
- r w x r - x r - -
│ │   │ │   │ │   │
│ └───┘ └───┘ └───┘
│  │     │     │
│  │     │     └── others  (everyone else)
│  │     └──────── group   (the "dev" group)
│  └────────────── owner   (shadows)
│
└── file type:  - = regular file
                d = directory
                l = symbolic link
```

> [!note] 
 > Symbolic link permissions are generally ignored on Linux. Access is controlled by the permissions of the file or directory the link points to.

Each group of three characters means the same thing:

| Character | Meaning for files          | Meaning for directories             |
| --------- | -------------------------- | ----------------------------------- |
| `r`       | Can read the file          | Can list the directory contents     |
| `w`       | Can write/modify the file  | Can create or delete files inside   |
| `x`       | Can execute (run) the file | Can enter (`cd` into) the directory |
| `-`       | Permission denied          | Permission denied                   |
> [!note]
> **Important:** Directory permissions work differently from file permissions.
>
> For directories:
>
> - `r` = list the contents (`ls`)
> - `w` = create, delete, or rename files inside
> - `x` = enter the directory (`cd`)
>
> A directory usually needs both `r` and `x` to be useful.
---

## Worked example

```
-rwxr-xr--
```

Break it into four parts:

```
-    rwx    r-x    r--
│     │      │      │
│     │      │      └── others: read only
│     │      └───────── group:  read + execute
│     └──────────────── owner:  read + write + execute
└──────────────────────  type:  regular file
```

So `shadows` can read, write, and run this file. Anyone in the `dev` group can read and run it, but not modify it. Everyone else can only read it.

---

## Changing permissions with `chmod`

`chmod` stands for **change mode**. There are two ways to use it.

---

### Symbolic mode — easier to read

```bash
# Give the owner execute permission
chmod u+x script.sh

# Remove write permission from the group
chmod g-w notes.txt

# Set others to read-only
chmod o=r file.txt

# Give everyone execute permission
chmod a+x program

# Multiple changes at once
chmod u+x,g-w,o=r deploy.sh
```

The letters:

| Letter | Who        |
| ------ | ---------- |
| `u`    | user/owner |
| `g`    | group      |
| `o`    | others     |
| `a`    | all three  |

The operators:

| Operator | Meaning              |
| -------- | -------------------- |
| `+`      | add a permission     |
| `-`      | remove a permission  |
| `=`      | set exactly (replace)|

---

### Octal mode — faster once you learn it

Each permission maps to a number:

```
r = 4
w = 2
x = 1
- = 0
```

Add them up for each group of three:

```
rwx = 4+2+1 = 7
rw- = 4+2+0 = 6
r-x = 4+0+1 = 5
r-- = 4+0+0 = 4
--- = 0+0+0 = 0
```

Then put all three numbers together:

```bash
chmod 755 script.sh
#     │││
#     ││└── others:  r-x = 5
#     │└─── group:   r-x = 5
#     └──── owner:   rwx = 7
# Result: rwxr-xr-x
```

> [!note]
> Applying `755` recursively is often not ideal because it gives execute permission to every file.
>
> In practice, directories and files are often assigned different permissions.

**The most common values:**

| Octal | String      | Typical use                              |
| ----- | ----------- | ---------------------------------------- |
| `755` | `rwxr-xr-x` | Scripts, programs — owner edits, all run |
| `644` | `rw-r--r--` | Regular files — owner edits, all read    |
| `600` | `rw-------` | Private files — SSH keys, secrets        |
| `700` | `rwx------` | Private scripts — only you can run them  |
| `777` | `rwxrwxrwx` | Full access for everyone (avoid this)    |
> [!warning]
> `777` gives everyone full control of a file or directory.
>
> It is usually a sign that permissions are being used incorrectly and should almost never be required.

> **Quick test:** What is `chmod 764`?
>
> `7 = rwx` (owner), `6 = rw-` (group), `4 = r--` (others)
>
> Result: `rwxrw-r--`

---

## Recursive permissions

To change permissions on a directory and everything inside it:

```bash
# Apply 755 to a directory and all its contents
chmod -R 755 myproject/
```

Use `-R` carefully — it changes permissions on every single file and subfolder.

> [!warning]
> Recursive permission changes are common but can be dangerous.
>
> `chmod -R 755` gives execute permission to every file inside the directory, even files that are not programs.
>
> In production systems, files and directories are often assigned different permissions.

---

## Checking permissions

```bash
# Long list with permissions
ls -l file.txt

# Show permissions of a directory itself (not its contents)
ls -ld mydir/

# Human-readable permissions for a specific file
stat file.txt
```

Example:

```text
Access: (0644/-rw-r--r--)
```

This shows both the octal (`644`) and symbolic (`rw-r--r--`) forms of the permissions.

---

## Real-world examples

```bash
# You downloaded a script but can't run it — give yourself execute permission
chmod u+x downloaded-script.sh
./downloaded-script.sh

# Lock down your SSH private key (required by SSH — it refuses to use a key that's too open)
chmod 600 ~/.ssh/id_ed25519  
# or  
chmod 600 ~/.ssh/id_rsa

# A web server needs to serve files — set read for all
chmod 644 /var/www/html/index.html

# A deployment script — only you can modify, everyone can run
chmod 755 deploy.sh

# Make a script executable
chmod +x script.sh

# Run it  
./script.sh
```

---
## Quick practice

Create a file and experiment with permissions:

```bash
touch test.sh

chmod 644 test.sh
ls -l test.sh

chmod u+x test.sh
ls -l test.sh

chmod 755 test.sh
ls -l test.sh
```

Observe how the permission string changes after each command.

---
## Summary

```
Reading permissions
────────────────────────────────────────
  -rwxr-xr--
  │ │   │   └── others
  │ │   └────── group
  │ └────────── owner
  └──────────── type (- file, d dir, l link)
  
────────────────────────────────────────  

 # Make a script executable
 chmod +x script.sh

────────────────────────────────────────
  chmod symbolic
────────────────────────────────────────
  chmod +x file     make executable
  chmod u+x file    add execute for owner
  chmod g-w file    remove write for group
  chmod o=r file    set others to read only
  chmod a+r file    add read for everyone

chmod octal
────────────────────────────────────────
  r=4  w=2  x=1
  chmod 755   →  rwxr-xr-x
  chmod 644   →  rw-r--r--
  chmod 600   →  rw-------
```

---

## What's next

Now that you understand what permissions are, you need to understand who *owns* the files — users, groups, and how they relate to each other.

➡️ [05 · Users & Groups](./05-users-and-groups.md)

⬅️ [03 · Files & Directories](../chapter-01/03-files-and-dirs.md)
