# 09 · Processes

> **Chapter 3 of 3 — Power Tools**
> `intermediate` · ~11 min read

---

## What you'll learn

- What a process is and how Linux manages them
- How to view, find, and monitor running processes
- Foreground vs background processes
- How to send signals to stop, pause, or restart processes

---

## What is a process?

Every time you run a program — a command, a web server, a database — Linux creates a **process** for it. A process is a running instance of a program, with its own memory, CPU time, and identity.

Every process has:
- A **PID** (Process ID) — a unique number assigned at launch
- A **PPID** (Parent Process ID) — the PID of the process that started it
- An **owner** — the user it runs as
- A **state** — running, sleeping, stopped, zombie

```
init (PID 1)
├── systemd-journald (PID 312)
├── sshd (PID 841)
│   └── bash (PID 2041)         ← you SSH in, get a bash session
│       └── ls (PID 2198)       ← you run ls, bash spawns a child
└── nginx (PID 1103)
    ├── nginx worker (PID 1104)
    └── nginx worker (PID 1105)
```

Every process on the system is a child of PID 1 (init/systemd), which is the first process the kernel starts.

---

## Viewing processes

### `ps` — snapshot of running processes

```bash
# Show your own processes in this terminal
ps

# Show ALL processes from ALL users, with full detail
ps aux

# Show processes in a tree format
ps axjf

# Find a specific process by name
ps aux | grep nginx
```

The `ps aux` columns:

```
USER    PID  %CPU  %MEM    VSZ    RSS  TTY   STAT  START   TIME  COMMAND
alice  1842   0.0   0.1  17832   5912  pts/0  Ss   09:00   0:00  bash
alice  2041   0.0   0.0   9216   1024  pts/0  R+   10:05   0:00  ps aux
```

| Column   | Meaning                                      |
| -------- | -------------------------------------------- |
| `USER`   | Who owns the process                         |
| `PID`    | Process ID                                   |
| `%CPU`   | CPU usage percentage                         |
| `%MEM`   | Memory usage percentage                      |
| `STAT`   | State (S=sleeping, R=running, Z=zombie, T=stopped) |
| `COMMAND`| The command that was run                     |

---

### `top` — live process monitor

```bash
top
```

Shows a live, updating view of all processes. The most CPU-hungry processes appear at the top.

| Key in `top` | Action                              |
| ------------ | ----------------------------------- |
| `q`          | Quit                                |
| `k`          | Kill a process (enter PID)          |
| `M`          | Sort by memory usage                |
| `P`          | Sort by CPU usage                   |
| `u`          | Filter by username                  |
| `h`          | Help                                |

> **`htop`** is a modern, colourful, easier-to-use version of `top`. Install it with `sudo apt install htop`. Most people prefer it.

---

### `pgrep` — find a process by name

```bash
# Find the PID of nginx
pgrep nginx
# → 1103
# → 1104
# → 1105

# Find and show the process name too
pgrep -l python
# → 2048 python3

# Find processes owned by a specific user
pgrep -u www-data
```

---

## Foreground and background

By default, when you run a command, it runs in the **foreground** — your terminal waits for it to finish, and you can't type anything else.

### Running a process in the background

Add `&` at the end:

```bash
# Start a process in the background
sleep 60 &
# → [1] 2048
#    │   └── PID
#    └────── job number

# List background jobs in this session
jobs
# → [1]+  Running    sleep 60 &
```

### Moving between foreground and background

```bash
# Pause the current foreground process (send to background, stopped)
Ctrl+Z
# → [1]+  Stopped    ./long-running-script.sh

# Resume it in the background
bg %1

# Bring it back to the foreground
fg %1

# If there's only one job, you don't need the number
fg
```

### `nohup` — survive terminal close

If you run a process normally and close the terminal, the process dies. `nohup` prevents that:

```bash
# Run a script that keeps running after you log out
nohup ./backup.sh &

# Output goes to nohup.out by default
# Or specify your own:
nohup ./backup.sh > backup.log 2>&1 &
```

---

## Stopping processes

### `kill` — send a signal to a process

```bash
# Send the default signal (SIGTERM — politely ask to quit)
kill 2048

# Send a specific signal by number
kill -9 2048     # SIGKILL — force quit, cannot be ignored

# Send by name
kill -SIGTERM 2048
kill -SIGKILL 2048
```

### Common signals

| Signal    | Number | What it does                                          |
| --------- | ------ | ----------------------------------------------------- |
| `SIGTERM` | 15     | Politely ask the process to stop (default)            |
| `SIGKILL` | 9      | Force kill — immediate, cannot be ignored or caught   |
| `SIGHUP`  | 1      | Reload config (many daemons respond to this)          |
| `SIGINT`  | 2      | Interrupt (same as pressing `Ctrl+C`)                 |
| `SIGTSTP` | 20     | Pause (same as pressing `Ctrl+Z`)                     |

> **Use SIGTERM first, SIGKILL only if it won't stop.** SIGKILL bypasses all cleanup — the process can't save state or close files properly.

### `pkill` and `killall` — kill by name

```bash
# Kill all processes named "firefox"
pkill firefox

# Force kill all processes named "python3"
pkill -9 python3

# Kill all processes owned by bob
pkill -u bob

# killall is similar but requires an exact name match
killall nginx
```

---

## Practical scenarios

```bash
# A script is frozen and won't respond to Ctrl+C
# Find its PID and force-kill it
pgrep myscript
# → 3042
kill -9 3042

# A web server is using port 8080 and you need to know which process
lsof -i :8080
# → COMMAND   PID   USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
# → python3  3042  alice   3u  IPv4  12345    0t0    TCP *:8080 (LISTEN)

# Reload nginx config without restarting it
sudo kill -HUP $(pgrep nginx | head -1)

# Find all zombie processes (shouldn't have many)
ps aux | awk '$8=="Z" {print $2, $11}'
```

---

## Summary

```
Viewing processes
────────────────────────────────────────
  ps aux            all processes
  top / htop        live view
  pgrep name        find PID by name
  ps aux | grep x   search by name

Foreground / background
────────────────────────────────────────
  command &         start in background
  Ctrl+Z            pause to background
  bg %1             resume in background
  fg %1             bring to foreground
  jobs              list jobs
  nohup cmd &       survive terminal close

Stopping processes
────────────────────────────────────────
  kill PID          polite stop (SIGTERM)
  kill -9 PID       force kill (SIGKILL)
  pkill name        kill by process name
  Ctrl+C            interrupt current process
```

---

## Chapter 3 complete

Three chapters done. You can navigate the filesystem, manage permissions, chain commands, process text, and control running processes. One module left — taking everything you've learned and turning it into reusable shell scripts.

➡️ [Module 1 → 10 · Shell Scripting](../module-01/10-scripting.md)

⬅️ [08 · Text Tools](./08-text-tools.md)
