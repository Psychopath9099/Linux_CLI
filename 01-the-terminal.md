# 01 · The Terminal

> **Chapter 1 of 3 — The Shell & Filesystem**  
> `beginner` · ~10 min read

---

## What you'll learn

- What a terminal actually is (and what it isn't)
    
- The difference between the shell, the terminal, and the kernel
    
- How to open a terminal and what you're looking at
    
- Basic navigation around a prompt
    

---

## The big picture

Before typing a single command, it helps to know what each piece of the puzzle does.

```
┌─────────────────────────────────────────────────────┐
│                       YOU                           │
│              (typing commands)                      │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│               TERMINAL EMULATOR                     │
│   (the window on your screen — Konsole, GNOME       │
│    Terminal, xfce4-terminal, etc.)                  │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│                   THE SHELL                         │
│   (interprets what you type — often Bash)           │
│   Bash = Bourne Again Shell, a GNU project          │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│                  THE KERNEL                         │
│   (Linux — the actual OS core, written by           │
│    Linus Torvalds in 1991)                          │
└─────────────────────────────────────────────────────┘
```

**Think of it this way:**

- The **terminal emulator** is the window you interact with.
    
- The **shell** is the brain. It reads your text and figures out what to do.
    
- The **kernel** is the muscle. It does the actual work with hardware and files.
    

You talk to the shell. The shell talks to the kernel. The kernel talks to your computer.

> **Note:** Bash is the most common Linux shell, though others such as Zsh and Fish also exist.

---

## Opening a terminal

On most Linux desktops, one of these will work:

| Desktop Environment | Default Terminal         | Keyboard Shortcut |
| ------------------- | ------------------------ | ----------------- |
| GNOME               | gnome-terminal / Console | `Ctrl + Alt + T`  |
| KDE Plasma          | Konsole                  | `Ctrl + Alt + T`  |
| XFCE                | xfce4-terminal           | often `Super + T` |
| Any                 | Search "Terminal"        | varies            |
|                     |                          |                   |

Most users interact through a terminal emulator inside a desktop environment.

> **Note on "Super"** — the Linux community calls the Windows key the **Super key**. The name predates Microsoft Windows and doesn't belong to any one operating system.


---

## The prompt — what you're looking at

When the terminal opens, you'll see something like this:

```
shadows@ubuntu:~$
```

> **Note:** The hostname depends on your installation and Linux distribution.

Break it down:

```
shadows   @   ubuntu   :   ~   $
  │          │           │   │
  │          │           │   └── $ = regular user prompt
  │          │           └────── ~ = your home directory
  │          └────────────────── hostname (computer name)
  └───────────────────────────── username
```

>[!warning]
 > **The `$` vs `#` rule**
> 
> `$` means you are a regular user. This is where you should be most of the time.
> 
> `#` means you are **root** — the administrator account with unrestricted access to the system.
> 
> Beginners should avoid working as root unless a guide specifically requires it. A single mistake made as root can affect the entire system.
> 
> A single mistake made as root can affect the entire operating system.
> 
> We'll cover root and `sudo` properly in Chapter 2.

---

## Your first commands

Try these now. They can't break anything.

```bash
# What date and time is it?
date

# How long has the system been running?
uptime

# How much memory is available? (-h = human-readable)
free -h

# What's the current user?
whoami

# Show your current location
pwd

# Show the Linux kernel version
uname -r

# Close this terminal session
exit
```

```bash
# Open a command's manual page
# Press q to return to the terminal
man date
```

Manual pages (man pages) are the built-in documentation for Linux commands.

Example output for `free -h`:

```
               total        used        free
Mem:            15Gi        4.2Gi        9.8Gi
Swap:          2.0Gi        0.0Ki        2.0Gi
```

Don't worry about understanding every number yet. Just notice that flags like `-h` change how a command presents its output.

---

## Navigating the command line

You don't need to retype a command just because you made a typo.

|Key|What it does|
|---|---|
|`←` `→`|Move cursor left/right within the line|
|`↑` `↓`|Scroll through previous commands|
|`Ctrl + A`|Jump to the start of the line|
|`Ctrl + E`|Jump to the end of the line|
|`Ctrl + U`|Delete everything before the cursor|
|`Ctrl + C`|Cancel the current command|
|`Tab`|Autocomplete a command or filename|

> **Tab is your best friend.** Start typing a command or filename and press Tab. Bash will complete it if it can, or show you all the options if it can't. Learn this early and save yourself thousands of keystrokes.

---

## Copy and paste in the terminal

The usual `Ctrl + C` is taken (it cancels commands). Use these instead:

|Action|Shortcut|
|---|---|
|Copy|`Ctrl + Shift + C`|
|Paste|`Ctrl + Shift + V`|
|Copy (mouse)|Triple-click often selects an entire line in many terminal emulators, then middle-click to paste|

---

## Summary

```
What you learned
────────────────────────────────────────
  Terminal emulator → displays the shell
  Shell             → interprets commands (often Bash)
  Kernel            → manages hardware and system resources
  $                 → regular user prompt
  #                 → root prompt
  Tab               → autocomplete commands and files
```

---

## What's next

In the next lesson, we look at the filesystem — the directory tree that organises every file, every program, and every piece of hardware on your Linux system.

➡️ [02 · The Filesystem](02-filesystem.md)