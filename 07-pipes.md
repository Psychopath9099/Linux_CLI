# 07 · Pipes & Redirection

> **Chapter 3 of 3 — Power Tools**
> `intermediate` · ~13 min read

---

## What you'll learn

- The Unix philosophy of small, composable tools
- How the pipe `|` works
- Input/output redirection with `>`, `>>`, and `<`
- Handling stderr separately from stdout
- Building real pipelines that solve real problems

---

## The Unix philosophy

> *Write programs that do one thing and do it well. Write programs that work together.*
>
> — Doug McIlroy, Bell Labs, 1978

This is why Linux tools feel so different from most software. `ls` just lists files. `sort` just sorts lines. `grep` just filters lines. Each tool is deliberately limited — and that's the point.

The **pipe** is the mechanism that makes them powerful together.

---

## Every process has three streams

Before the pipe makes sense, you need to understand stdin, stdout, and stderr:

```
┌─────────────────────────────────────────────────────┐
│                    YOUR COMMAND                     │
│                                                     │
│  stdin (0)  ──────►  process  ──────►  stdout (1)  │
│  (keyboard)                          (your screen)  │
│                          │                          │
│                          └──────────►  stderr (2)  │
│                                      (also screen)  │
└─────────────────────────────────────────────────────┘
```

By default:
- `stdin` comes from your keyboard
- `stdout` goes to your screen
- `stderr` also goes to your screen

Redirection lets you change all three of those.

---

## The pipe `|`

The pipe takes the **stdout** of one command and feeds it as **stdin** to the next:

```bash
command1 | command2 | command3
```

```
ls /usr/bin  |  grep "python"  |  sort
     │               │               │
  list files    filter to lines    sort the
  in /usr/bin   containing          results
                "python"
```

```bash
# Count how many files are in /usr/bin
ls /usr/bin | wc -l
# → 1823

# Find all .conf files, then search inside them for "port"
find /etc -name "*.conf" | xargs grep -l "port"

# See the 10 biggest files in the current directory
du -sh * | sort -rh | head -10

# See which users are currently logged in (deduplicated and sorted)
who | awk '{print $1}' | sort | uniq
```

---

## Redirecting stdout `>`

Save a command's output to a file instead of the screen:

```bash
# Write output to a file (overwrites if file exists)
ls -la > filelist.txt

# Check the result
cat filelist.txt
```

> **Warning:** `>` overwrites without asking. If `filelist.txt` already has content, it's gone.

---

## Appending `>>`

Add to the end of a file instead of overwriting it:

```bash
# Log today's date to a running log
date >> activity.log

# Add system info
uptime >> activity.log

# See what accumulated
cat activity.log
# → Sun Jun  1 10:00:00 UTC 2025
# → 10:00:05 up 3 days,  2:14,  1 user,  load average: 0.08, 0.03, 0.01
```

---

## Redirecting stdin `<`

Feed a file as input to a command instead of the keyboard:

```bash
# Sort the contents of a file
sort < names.txt

# Same result, but most people write it this way (cat + pipe):
cat names.txt | sort
```

Both approaches work. The first is slightly more efficient (no `cat` process), but the second is easier to read and extend.

---

## Redirecting stderr `2>`

Errors are sent to **stderr** (stream 2), which is separate from **stdout** (stream 1). By default both go to your screen, which can be confusing.

```bash
# Separate errors into their own file
ls /nonexistent 2> errors.txt

# Discard errors completely
ls /nonexistent 2> /dev/null
#               ^^^^^^^^^^^ /dev/null is a black hole — anything written to it disappears

# Redirect both stdout AND stderr to the same file
ls /usr/bin /nonexistent > output.txt 2>&1
#                                     ^^^ "send stderr to where stdout is going"
```

---

## `/dev/null` — the discard pipe

`/dev/null` is a special file. Writing to it discards whatever you send. Reading from it gives you nothing.

```bash
# Run a noisy script but only care about errors
./long-script.sh > /dev/null

# Run something and suppress all output
./setup.sh > /dev/null 2>&1

# Generate empty input for testing
cat /dev/null
```

---

## Real pipelines in practice

This is where it gets genuinely useful. Here are real problems solved with pipelines:

```bash
# How many lines contain "ERROR" in the logs from the last boot?
journalctl -b | grep "ERROR" | wc -l

# Find the top 5 largest directories in /var
du -sh /var/* 2>/dev/null | sort -rh | head -5

# Which nginx config files reference port 443?
grep -r "443" /etc/nginx/ 2>/dev/null | cut -d: -f1 | sort -u

# See the most common IPs hitting your web server
cat /var/log/nginx/access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -10

# List all running processes, sort by memory usage, show top 10
ps aux | sort -k4 -rn | head -10
```

---

## `tee` — pipe AND save at the same time

`tee` is useful when you want to save output to a file but also still see it on screen:

```bash
# Show output on screen AND save it
./build.sh | tee build.log

# Append mode
./build.sh | tee -a build.log
```

```
./build.sh  ──► tee ──► screen (you see it)
                  │
                  └───► build.log (also saved)
```

---

## Summary

```
Streams
────────────────────────────────────────
  0 = stdin    (keyboard by default)
  1 = stdout   (screen by default)
  2 = stderr   (screen by default)

Redirection
────────────────────────────────────────
  cmd > file       stdout to file (overwrite)
  cmd >> file      stdout to file (append)
  cmd < file       file as stdin
  cmd 2> file      stderr to file
  cmd 2>/dev/null  discard errors
  cmd > f 2>&1     stdout + stderr to file

Pipes
────────────────────────────────────────
  cmd1 | cmd2          chain two commands
  cmd1 | cmd2 | cmd3   chain three
  cmd | tee file       output to screen AND file

Common pipeline idioms
────────────────────────────────────────
  ... | wc -l          count lines
  ... | sort -rn       sort numerically, reversed
  ... | uniq -c        count duplicates
  ... | head -10       show first 10
  ... | tail -10       show last 10
```

---

## What's next

Now you can chain commands. The next lesson gives you the tools to work with text — the core data type of the Linux universe.

➡️ [08 · Text Tools](./08-text-tools.md)

⬅️ [06 · sudo](../chapter-02/06-sudo.md)
