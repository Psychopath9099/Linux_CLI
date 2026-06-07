# 08 · Text Tools

> **Chapter 3 of 3 — Power Tools**
> `intermediate` · ~15 min read

---

## What you'll learn

- Search files with `grep`
- Transform text streams with `sed`
- Process structured text with `awk`
- Sort, deduplicate, count, and slice with `sort`, `uniq`, `wc`, `cut`

---

## Why text tools matter

In Linux, almost everything is stored as plain text — config files, logs, user databases, environment variables. The ability to search, transform, and extract from text turns a mass of raw data into actionable information.

These tools were written in the 1970s and 1980s. They're still used daily because nothing has ever replaced them.

---

## `grep` — search for patterns

`grep` prints every line in a file (or stream) that matches a pattern.

```bash
# Find lines containing "error" in a log file
grep "error" /var/log/syslog

# Case-insensitive search
grep -i "error" /var/log/syslog

# Show line numbers
grep -n "error" /var/log/syslog

# Show 2 lines of context before and after each match
grep -C 2 "error" /var/log/syslog

# Count how many lines match
grep -c "error" /var/log/syslog

# Invert — show lines that DON'T match
grep -v "DEBUG" app.log

# Search recursively in a directory
grep -r "TODO" ~/projects/

# List only the filenames that contain a match (not the lines themselves)
grep -rl "password" /etc/
```

### Using regex in grep

```bash
# Lines that start with "alice"
grep "^alice" /etc/passwd

# Lines that end with "bash"
grep "bash$" /etc/passwd

# Lines containing any digit
grep "[0-9]" data.txt

# Extended regex (gives you +, ?, |, and groups without backslashes)
grep -E "error|warning|critical" /var/log/syslog
```

---

## `sed` — stream editor

`sed` reads text line by line and applies transformations. The most common use is **find and replace**.

```bash
# Replace "foo" with "bar" on every line
sed 's/foo/bar/g' file.txt
#   ─────────────
#   s = substitute
#   g = global (replace all occurrences per line, not just the first)

# Replace only on lines containing "alice"
sed '/alice/s/admin/user/g' file.txt

# Delete lines matching a pattern
sed '/^#/d' config.txt     # delete all comment lines (starting with #)
sed '/^$/d' file.txt       # delete all blank lines

# Edit a file in-place (modifies the original file)
sed -i 's/localhost/127.0.0.1/g' app.conf

# Edit in-place with a backup
sed -i.bak 's/localhost/127.0.0.1/g' app.conf
# creates app.conf.bak before modifying
```

### Practical sed examples

```bash
# Remove trailing whitespace from every line
sed 's/[[:space:]]*$//' file.txt

# Add a line number prefix to each line
sed '=' file.txt | sed 'N;s/\n/\t/'

# Print only lines 5 to 10
sed -n '5,10p' file.txt

# Print every line except lines 5 to 10
sed '5,10d' file.txt
```

---

## `awk` — field processing

`awk` is a tiny language built for working with structured text — think CSV, `/etc/passwd`, log files. It processes one line at a time and lets you work with individual fields.

```bash
# Print the first field of each line (default separator = whitespace)
awk '{print $1}' file.txt

# Print the first and third fields
awk '{print $1, $3}' file.txt

# Use a different separator (-F = field separator)
awk -F: '{print $1}' /etc/passwd        # print all usernames
awk -F: '{print $1, $3}' /etc/passwd    # username and UID

# Print only lines where field 3 is greater than 1000 (regular users)
awk -F: '$3 >= 1000 {print $1}' /etc/passwd

# Sum the values in the second column
awk '{sum += $2} END {print sum}' numbers.txt

# Print a header and process lines
awk 'BEGIN {print "Users:"} {print $1}' /etc/passwd
```

### `awk` special variables

| Variable | Meaning                                    |
| -------- | ------------------------------------------ |
| `$0`     | The entire current line                    |
| `$1`     | First field                                |
| `$2`     | Second field (and so on)                   |
| `NF`     | Number of fields in the current line       |
| `NR`     | Current line number (total lines read)     |
| `FS`     | Field separator (default: whitespace)      |
| `OFS`    | Output field separator (default: space)    |

```bash
# Print the last field of every line
awk '{print $NF}' file.txt

# Print the line number and content
awk '{print NR": "$0}' file.txt
```

---

## `sort` — sort lines

```bash
# Sort lines alphabetically
sort file.txt

# Sort in reverse
sort -r file.txt

# Sort numerically
sort -n numbers.txt

# Sort by the second field (space-separated)
sort -k2 data.txt

# Sort by second field, numerically, reversed
sort -k2 -rn data.txt

# Sort and remove duplicates
sort -u file.txt

# Sort human-readable sizes (1K, 5M, 2G)
du -sh * | sort -h
```

---

## `uniq` — find or remove duplicates

`uniq` only removes **adjacent** duplicates — so almost always used after `sort`:

```bash
# Remove duplicate lines
sort file.txt | uniq

# Count how many times each line appears
sort file.txt | uniq -c

# Show only lines that appear more than once
sort file.txt | uniq -d

# Show only lines that appear exactly once
sort file.txt | uniq -u
```

---

## `wc` — count things

```bash
# Count lines, words, and bytes
wc file.txt
# → 42 183 1024 file.txt

# Count only lines
wc -l file.txt

# Count only words
wc -w file.txt

# Count files in a directory
ls | wc -l
```

---

## `cut` — extract specific columns

```bash
# Get the first field, using colon as delimiter
cut -d: -f1 /etc/passwd

# Get fields 1 and 3
cut -d: -f1,3 /etc/passwd

# Get a range of fields
cut -d: -f1-4 /etc/passwd

# Get characters 1-10 from each line
cut -c1-10 file.txt
```

---

## Putting it all together

These tools become truly powerful in combination. Here are some real examples:

```bash
# How many unique users are in the system (not system accounts)?
awk -F: '$3 >= 1000 {print $1}' /etc/passwd | sort | uniq | wc -l

# Find the 5 most common error messages in a log
grep -i "error" /var/log/syslog \
  | awk '{$1=$2=$3=""; print $0}' \
  | sort | uniq -c | sort -rn | head -5

# Extract all IP addresses from a log file
grep -oE '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' access.log | sort | uniq -c | sort -rn

# List all config files that have been modified today
find /etc -name "*.conf" -mtime -1 | xargs ls -lt | awk '{print $NF}'

# Count lines of code by file extension
find . -type f | grep -E "\.(py|js|sh)$" | xargs wc -l | sort -rn | head -20
```

---

## Summary

```
grep — search
────────────────────────────────────────
  grep "pattern" file       basic search
  grep -i                   case insensitive
  grep -r                   recursive
  grep -v                   invert (lines that don't match)
  grep -c                   count matches
  grep -E "a|b"             extended regex

sed — transform
────────────────────────────────────────
  sed 's/old/new/g' file    replace all occurrences
  sed '/pattern/d' file     delete matching lines
  sed -i 's/a/b/g' file     edit in-place

awk — fields
────────────────────────────────────────
  awk '{print $1}' file     first field
  awk -F: '{print $1}'      custom separator
  awk '$3 > 100 {print}'    conditional
  awk '{sum+=$1} END{print sum}'  accumulate

sort / uniq / wc / cut
────────────────────────────────────────
  sort -rn file             reverse numeric sort
  sort | uniq -c            count duplicates
  wc -l file                count lines
  cut -d: -f1 file          first colon-delimited field
```

---

## What's next

The last tool chapter — processes. Running programs, stopping them, putting them in the background, understanding signals.

➡️ [09 · Processes](./09-processes.md)

⬅️ [07 · Pipes & Redirection](./07-pipes.md)
