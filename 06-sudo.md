# 06 · sudo

> **Chapter 2 of 3 — Permissions & Users**
> `beginner → intermediate` · ~10 min read

---

## What you'll learn

- What `sudo` is and why it exists
- How to use it safely
- The difference between `sudo` and `su`
- What you should never do as root

---

## The problem `sudo` solves

Most things you do on a Linux system don't require special permissions. But some things do — installing software, editing system configuration files, managing users, changing network settings.

You could just log in as the **root** user and do everything as root. But that's dangerous. One bad command, one wrong path, and you could delete something critical with no warning.

`sudo` (**su**peruser **do**) is the solution. It lets you run a single command as root, then drops back to your normal user immediately. You get the power you need, only when you need it.

```
Regular session                      With sudo
─────────────────────────────────    ────────────────────────────────
alice@ubuntu:~$                      alice@ubuntu:~$
  ↓ all commands run as alice          ↓ this command runs as root
  ↓ no system-level access             sudo apt update
  ↓ no installing packages             ↓ done, back to alice
                                       alice@ubuntu:~$
```

---

## Basic usage

```bash
# Run a single command as root
sudo apt update

# Edit a system file (you can't write to /etc as a regular user)
sudo nano /etc/hosts

# Run a command as a specific user (not root)
sudo -u www-data ls /var/www

# Start a root shell (be careful)
sudo -i

# Start a root shell (alternative)
sudo su
```

When you run `sudo`, it will ask for **your own password** (not root's). It then grants you elevated access for a short window (usually 15 minutes) before asking again.

---

## Who can use sudo?

Not every user on the system can use `sudo`. Only users in the **sudo** group (on Ubuntu/Debian) or **wheel** group (on Fedora/RHEL) are allowed.

```bash
# Check if you can sudo
sudo whoami
# → root  (if allowed)

# Check what sudo privileges you have
sudo -l

# Add a user to the sudo group
sudo usermod -aG sudo alice
```

The configuration lives in `/etc/sudoers`. Never edit this file directly — use `visudo` instead, which validates the syntax before saving:

```bash
sudo visudo
```

---

## `sudo` vs `su`

These are different commands that people often confuse:

| Command      | What it does                                              |
| ------------ | --------------------------------------------------------- |
| `sudo cmd`   | Run one command as root, using *your* password            |
| `su`         | Switch to root user entirely, using *root's* password     |
| `su - alice` | Switch to alice's account, using alice's password         |
| `sudo su`    | Switch to root using your own password (not root's)       |
| `sudo -i`    | Open a full root login shell, using your password         |

On modern Ubuntu and Debian systems, the root account has no password set by default. You can't `su` to root directly — you must go through `sudo`. This is intentional.

---

## What root can do (that you can't)

Root has no restrictions. Zero. It can:

```bash
# Delete the entire filesystem (DO NOT RUN THIS)
rm -rf /

# Write to any file, including system files
echo "0.0.0.0 google.com" >> /etc/hosts

# Kill any process, including system processes
kill -9 1   # PID 1 is init — this would crash the system

# Read any file, including other users' private keys
cat /home/bob/.ssh/id_rsa

# Install or remove software
apt install nginx
apt remove --purge nginx
```

The terminal does not protect you from yourself as root. It will execute exactly what you type, with no confirmation, no second chance, no undo.

---

## Good habits with sudo

**Do:**

```bash
# Run only the specific command that needs root
sudo apt update
sudo systemctl restart nginx
sudo chmod 755 /var/www/html

# Use sudo -l to check what you're allowed to do
sudo -l

# Exit a root shell immediately when done
sudo -i
# ... do what you need ...
exit
```

**Avoid:**

```bash
# Avoid staying in a root shell for routine work
sudo -i
# ... browsing files, running scripts, being generally reckless ...

# Avoid running unknown scripts with sudo
sudo curl https://sketchy-site.com/install.sh | bash
#          ^^^ you have no idea what this does

# Avoid giving sudo to everyone
sudo usermod -aG sudo untrusteduser
```

---

## Logs — sudo keeps records

Every `sudo` command is logged. If something goes wrong on a shared system, there's a record of who ran what and when.

```bash
# On Ubuntu/Debian, sudo logs go here
sudo grep sudo /var/log/auth.log

# You'll see entries like:
# Jun  1 10:05:43 ubuntu sudo: alice : TTY=pts/0 ; PWD=/home/alice ;
#   USER=root ; COMMAND=/usr/bin/apt update
```

This is a feature. If you're the only person on your machine, it mostly means you can audit your own past actions.

---

## Summary

```
Core usage
────────────────────────────────────────
  sudo command           run command as root
  sudo -i                open root shell
  sudo -l                list your permissions
  sudo -u user command   run as specific user

Who can sudo
────────────────────────────────────────
  sudo group (Ubuntu/Debian)
  wheel group (Fedora/RHEL)
  sudo usermod -aG sudo alice

Rules of thumb
────────────────────────────────────────
  ✓ use sudo only for the command that needs it
  ✓ exit root shells immediately when done
  ✗ don't run unknown scripts as sudo
  ✗ don't stay in a root shell during normal work
```

---

## Chapter 2 complete

You now understand the full permission model: what permission strings mean, how `chmod` works, who owns files, and how to safely escalate privileges. Chapter 3 is where things get genuinely powerful.

➡️ [Chapter 3 → 07 · Pipes & Redirection](../chapter-03/07-pipes.md)

⬅️ [05 · Users & Groups](./05-users-and-groups.md)
