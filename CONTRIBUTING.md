# Contributing to LinuxLore

First — thank you. This guide exists to help people, and every improvement makes it better for the next person who lands here confused.

---

## What you can contribute

- **Typo or grammar fixes** — always welcome, no discussion needed
- **Clearer explanations** — if something confused you, it will confuse others
- **Better examples** — practical, real-world examples beat abstract ones
- **New lessons** — must follow the existing style and difficulty curve
- **Corrections** — if something is technically wrong, please open an issue first

---

## How to contribute

1. **Fork** the repository
2. **Clone** your fork locally
   ```bash
   git clone https://github.com/yourusername/linux-lore.git
   cd linux-lore
   ```
3. **Create a branch** with a descriptive name
   ```bash
   git checkout -b fix/typo-lesson-03
   git checkout -b improve/pipes-examples
   git checkout -b add/lesson-11-cron
   ```
4. **Make your changes**
5. **Commit** with a clear message
   ```bash
   git commit -m "fix: correct chmod octal example in lesson 04"
   git commit -m "improve: add real-world grep examples to lesson 08"
   ```
6. **Push** and open a pull request

---

## Style guide

**Tone:** Plain English. Write like you are explaining to a smart friend who has never used a terminal. Avoid jargon without explanation. No condescension.

**Code blocks:** Always use fenced code blocks with `bash` as the language tag.

```bash
# Good — includes a comment explaining what it does
chmod 755 script.sh

# Avoid — no context
chmod 755 script.sh
```

**Comments in code:** Add `# comments` to explain non-obvious parts of commands. Keep them short.

**Examples:** Prefer real, runnable examples over abstract placeholders. `grep "error" /var/log/syslog` beats `grep "pattern" file`.

**Warnings:** Use blockquotes for important warnings.

> ⚠️ This is a warning about something that can cause real damage.

**Structure:** Every lesson should have:
- A brief intro (what and why)
- Concept explanation with a diagram or table if helpful
- Code examples with comments
- A summary block at the end
- Links to previous and next lessons

---

## Reporting issues

Use the issue templates provided in `.github/ISSUE_TEMPLATE/`:

- **Bug report** — for incorrect commands, broken links, or factual errors
- **Content suggestion** — for new topics, missing explanations, or lesson ideas

---

## Code of conduct

Be kind. This project exists to help beginners. Condescending comments, gatekeeping, and hostile reviews are not welcome and will be removed.

---

Thank you for helping make Linux less intimidating.
