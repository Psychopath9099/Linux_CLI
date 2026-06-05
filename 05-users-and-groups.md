# 05 · Users & Groups

> **Chapter 2 of 3 — Permissions & Users**
> `beginner → intermediate` · ~11 min read

---

## What you'll learn

- What users and groups are in Linux
- How to check who you are and what groups you belong to
- How to change file ownership with `chown`
- How to add users and manage groups

---

## Users in Linux

Every file is owned by a **user**. Every process runs as a **user**. This is how Linux tracks who did what and enforces permissions.

There are three kinds of users:

| Type            | Description                                                            |
| --------------- | ---------------------------------------------------------------------- |
| **root**        | The superuser. UID 0. No restrictions. Can do anything on the system.  |
| **Regular user**| You. Created at install time. Has a home directory in `/home/`.        |
| **System user** | Not a real person. Created by programs like `www-data` for nginx, `postgres` for a database server. Used to run services with limited permissions. |

---

## Checking your identity

```bash
# Who am I?
whoami
# → alice

# My full identity — UID, GID, and all groups
id
# → uid=1000(alice) gid=1000(alice) groups=1000(alice),27(sudo),999(docker)

# Who else is logged in right now?
who
# → alice    pts/0        Jun  1 09:00 (192.168.1.5)
```

---

## Groups

A **group** is a collection of users. Groups let you give the same permissions to multiple users without changing each file individually.

Every user has:
- A **primary group** (usually the same name as their username)
- Zero or more **secondary groups** (extra groups for additional access)

```bash
# Show all groups you belong to
groups
# → alice sudo docker audio

# Show groups for a specific user
groups bob
# → bob : bob sudo
```

If you want to access a Docker socket, a USB device, or a particular directory — often the solution is adding your user to the right group.

---

## The `/etc/passwd` file

Every user account is stored in `/etc/passwd`. Don't let the name fool you — passwords aren't actually in here anymore (they're in `/etc/shadow`).

```bash
cat /etc/passwd
```

```
root:x:0:0:root:/root:/bin/bash
alice:x:1000:1000:Alice Smith:/home/alice:/bin/bash
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
```

Each line has seven fields separated by `:`:

```
alice : x   : 1000 : 1000 : Alice Smith : /home/alice : /bin/bash
  │      │      │      │        │              │             │
  │      │      │      │        │              │             └── login shell
  │      │      │      │        │              └──────────────── home directory
  │      │      │      │        └─────────────────────────────── full name (optional)
  │      │      │      └──────────────────────────────────────── primary GID
  │      │      └─────────────────────────────────────────────── UID
  │      └────────────────────────────────────────────────────── password (x = in /etc/shadow)
  └───────────────────────────────────────────────────────────── username
```

---

## Changing file ownership with `chown`

`chown` stands for **change owner**.

```bash
# Change the owner of a file
sudo chown bob file.txt

# Change owner AND group
sudo chown bob:developers file.txt

# Change just the group (note the colon at the start)
sudo chown :developers file.txt

# Apply recursively to a directory
sudo chown -R www-data:www-data /var/www/html
```

> `chown` almost always requires `sudo` — you can only give away ownership of something you own, and most system files belong to root.

---

## Managing users

These commands require `sudo` (covered in the next lesson):

```bash
# Create a new user (and their home directory)
sudo useradd -m -s /bin/bash bob

# Set or change a user's password
sudo passwd bob

# Delete a user
sudo userdel bob

# Delete a user AND their home directory
sudo userdel -r bob

# Modify a user's settings (e.g. change their shell)
sudo usermod -s /bin/zsh alice
```

---

## Managing groups

```bash
# Create a new group
sudo groupadd developers

# Add a user to a group (takes effect on next login)
sudo usermod -aG developers alice
#                 │
#                 └── -a = append (don't remove existing groups)

# Remove a user from a group
sudo gpasswd -d alice developers

# Delete a group
sudo groupdel developers

# See all groups on the system
cat /etc/group
```

---

## Practical scenario

Your web server (`nginx`) runs as the `www-data` user. You want to allow nginx to serve files from your project directory, and you also want to edit those files as `alice`.

```bash
# 1. Create a shared group for this project
sudo groupadd webteam

# 2. Add alice and www-data to the group
sudo usermod -aG webteam alice
sudo usermod -aG webteam www-data

# 3. Set the project directory ownership
sudo chown -R alice:webteam /var/www/myproject

# 4. Set permissions — group can read+execute directories, read files
sudo chmod -R 750 /var/www/myproject

# Now alice can manage files, and nginx can serve them
```

---

## Summary

```
Checking identity
────────────────────────────────────────
  whoami              your username
  id                  uid, gid, all groups
  groups              list your groups

Ownership
────────────────────────────────────────
  chown user file     change owner
  chown user:grp file change owner and group
  chown -R user dir   recursive

User management (requires sudo)
────────────────────────────────────────
  useradd -m -s /bin/bash bob    create user
  passwd bob                     set password
  usermod -aG group user         add to group
  userdel -r bob                 delete user
```

---

## What's next

Both `chown` and many `useradd` commands need `sudo`. Let's talk about what sudo actually is, how to use it safely, and what root can do that you can't.

➡️ [06 · sudo](./06-sudo.md)

⬅️ [04 · Permissions](./04-permissions.md)
