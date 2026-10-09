# AcoeAI Custom Commands

## What Is a Custom Command? (in one sentence)

> A custom command is a **saved prompt** you run by typing `/name`.

Instead of typing the same long instructions every day, you write them down **once**,
save them in a file, and then just type a short word like `/todo` to use them again.

Think of it like a **text shortcut** for your AI assistant.

```
Without a custom command:
  you type 40 lines of instructions every single time

With a custom command:
  you type  /todo  and press enter
```

AcoeAI already has built-in commands like `/help`, `/undo`, `/redo`, `/share`, and
`/init`. Custom commands are **your own extra commands** on top of those.

---

## How to Read the Examples in This Guide

Every example follows the same friendly pattern so it's easy to follow:

| Part | Meaning |
| --- | --- |
| **What it does** | A plain-English description |
| **📄 The file** | Where you create it and what you write inside |
| **▶️ How to run it** | What you type in the UI |
| **✅ What happens** | The result you should expect |

---

# Part 1 — The Basics

A command is just a **Markdown file** placed in a special folder.
The **file name becomes the command name**.

If you create this file:

```
.acoeai/commands/todo.md
```

…then this command becomes available:

```
/todo
```

Inside the file you can optionally add a small header (called **frontmatter**)
between two lines of `---`. It's used for settings like a short description.

Here is the smallest useful command you can make:

**📄 `.acoeai/commands/todo.md`**

```markdown
---
description: Add an item to my todo list
---

Read my current todo list in @TODO.md, then add a new task at the bottom.
Keep the existing format and don't reorder anything.
```

**▶️ How to run it**

```
/todo
```

**✅ What happens**
AcoeAI reads your `TODO.md` file, adds a new task at the bottom, and keeps the
format the same.

That's it — you just made your first command. Everything below builds on this idea.

---

# Part 2 — Making Prompts Dynamic

A saved prompt is useful, but it becomes **powerful** when it can accept input.
Three simple tools let you do that.

## 2.1 — `$ARGUMENTS` : capture everything the user typed

Use `$ARGUMENTS` as a placeholder for **all words** typed after the command.

**📄 `.acoeai/commands/translate.md`**

```markdown
---
description: Translate text into plain English
---

Translate the following into simple English, then give a one-line summary:
$ARGUMENTS
```

**▶️ How to run it**

```
/translate Le chat noir dort sur le tapis rouge
```

**✅ What happens**
`$ARGUMENTS` becomes `Le chat noir dort sur le tapis rouge`, so AcoeAI translates
that sentence and then summarizes it in one line.

> 💡 **Good to know:** Everything after the command name is treated as the arguments —
> you don't need special quotes for normal sentences.

## 2.2 — `$1`, `$2`, `$3` : pick individual words

Sometimes you don't want *all* the text — you want **specific positions**.
Use `$1` for the first argument, `$2` for the second, and so on.

**📄 `.acoeai/commands/meeting-note.md`**

```markdown
---
description: Add a meeting note to a file
---

Open the file named $1.
Add a new note with the title "$2" and the summary "$3".
Keep the newest note at the top.
```

**▶️ How to run it**

```
/meeting-note notes.md "Budget Review" "We agreed to cut costs by 10%"
```

**✅ What happens**

| Placeholder | Filled with |
| --- | --- |
| `$1` | `notes.md` |
| `$2` | `Budget Review` |
| `$3` | `We agreed to cut costs by 10%` |

> 💡 **Tip:** Wrap arguments containing spaces in quotes so they stay together.

## 2.3 — `` !`command` `` : inject live shell output

Want your prompt to include **real, current information** like the time, a file list,
or git history? Wrap a shell command in an exclamation mark and backticks.

The command runs in your **project root folder**, and its output is pasted into the
prompt automatically.

**📄 `.acoeai/commands/what-changed.md`**

```markdown
---
description: Explain what I changed recently
---

Here are my most recent commits:
!`git log --oneline -5`

Explain what these changes mean in plain English.
```

**▶️ How to run it**

```
/what-changed
```

**✅ What happens**
Your last 5 commit messages are inserted into the prompt, and AcoeAI explains them
like a friendly changelog.

> 💡 **Good to know:** Use this for anything live — `!`date``, `!`ls -la``, `!`docker ps``.

## 2.4 — `@filename` : include a file's contents

Use `@` followed by a file path to **pull a file into the prompt**.

**📄 `.acoeai/commands/summarize.md`**

```markdown
---
description: Summarize a text file
---

Read @meeting-notes.txt and summarize it in 5 bullet points.
Then list any action items you can find.
```

**▶️ How to run it**

```
/summarize
```

**✅ What happens**
The full contents of `meeting-notes.txt` are added to the prompt, and you get a short
summary plus any to-dos.

> 💡 **Combine them!** You can use `$1`, `!`shell``, and `@file` all in one command for
> serious power. See Example 7 below.

---

# Part 3 — Ten Simple, Real Examples

Each one is self-contained. Pick the ones you like.

---

### Example 1 — `/readme` : write a README from scratch

**What it does** Turns a bare code folder into a friendly README.

**📄 `.acoeai/commands/readme.md`**

```markdown
---
description: Generate a README for the project
---

Look at the files in this project.
Write a README.md that explains:
- what the project does
- how to install it
- how to run it
Use simple language a beginner can follow.
```

**▶️ `/readme`**

**✅** A new `README.md` appears, written for non-experts.

---

### Example 2 — `/email` : turn rough notes into a polite email

**What it does** Converts bullet points into a ready-to-send message.

**📄 `.acoeai/commands/email.md`**

```markdown
---
description: Draft a polite email from rough notes
---

Turn the following rough notes into a short, polite email.
Add a subject line and a friendly closing.

Notes:
$ARGUMENTS
```

**▶️ `/email need day off friday, dentist appointment, back monday`**

**✅** A short professional email with a subject, body, and sign-off.

---

### Example 3 — `/explain-circle` : explain a file to a beginner

**What it does** Explains any file as if talking to a curious beginner.

**📄 `.acoeai/commands/explain-circle.md`**

```markdown
---
description: Explain a file in beginner-friendly language
---

Explain @$1 as if the reader has never seen this kind of file before.
Use short sentences and a small analogy.
Finish with one "why it matters" line.
```

**▶️ `/explain-circle src/helpers.js`**

**✅** A simple, analogy-based explanation of `helpers.js`.

---

### Example 4 — `/fix-spelling` : correct spelling in a file

**What it does** Fixes typos without touching the meaning.

**📄 `.acoeai/commands/fix-spelling.md`**

```markdown
---
description: Fix spelling and grammar in a file
---

Open @$1 and fix only spelling and grammar mistakes.
Do not change the meaning, formatting, or code blocks.
Show me a short list of what you corrected.
```

**▶️ `/fix-spelling docs/guide.md`**

**✅** The file is cleaned up, and you get a list of the fixes.

---

### Example 5 — `/todo-scan` : find all TODO comments in code

**What it does** Collects every `TODO` note scattered across your code.

**📄 `.acoeai/commands/todo-scan.md`**

```markdown
---
description: Collect all TODO comments in the project
---

Search the project for comments containing TODO or FIXME.
List each one as: file -> line -> the note.
Group them by folder so they are easy to skim.
```

**▶️ `/todo-scan`**

**✅** A grouped list of every unfinished note in your codebase.

---

### Example 6 — `/commit` : write a commit message

**What it does** Drafts a clean commit message from your staged changes.

**📄 `.acoeai/commands/commit.md`**

```markdown
---
description: Draft a commit message for staged changes
---

Here is what I have staged:
!`git diff --cached --stat`

Write a short commit message in the format:
type(scope): summary

Then add one sentence explaining why.
```

**▶️ `/commit`**

**✅** A ready-to-use commit message like `fix(auth): handle expired tokens`.

---

### Example 7 — `/standup` : prepare a daily stand-up update

**What it does** This one combines **shell output** with a clear, structured prompt.

**📄 `.acoeai/commands/standup.md`**

```markdown
---
description: Prepare my stand-up update
---

Yesterday's commits:
!`git log --since=yesterday --oneline`

Uncommitted work:
!`git status --short`

Write a stand-up update with three sections:
1. Done
2. Doing
3. Blocked
Keep each section to a maximum of three bullets.
```

**▶️ `/standup`**

**✅** A tidy stand-up note with Done / Doing / Blocked sections.

---

### Example 8 — `/regex` : build a regular expression

**What it does** Creates and *explains* a regex, which is normally the confusing part.

**📄 `.acoeai/commands/regex.md`**

```markdown
---
description: Write and explain a regular expression
---

I need a regular expression that matches: $ARGUMENTS

Give me:
1. The regex itself
2. A plain-English explanation of each part
3. Two matching examples and two non-matching examples
```

**▶️ `/regex an email address`**

**✅** A working regex with notes explaining every piece.

---

### Example 9 — `/cleanup` : tidy a messy data file

**What it does** Reformat one file to match another file's style.

**📄 `.acoeai/commands/cleanup.md`**

```markdown
---
description: Reformat a file to match another file's style
---

Use @$1 as the style reference.
Reformat the file @$2 so it matches that style.
Do not change any actual values or content.
Tell me the main differences you applied.
```

**▶️ `/cleanup templates/clean.csv data/messy.csv`**

**✅** `messy.csv` now follows the same style as `clean.csv`.

---

### Example 10 — `/note` : save a quick idea

**What it does** Appends a timestamped idea to your notes file.

**📄 `.acoeai/commands/note.md`**

```markdown
---
description: Save a timestamped note
---

Today's date and time is:
!`date`

Add the following idea to the bottom of @ideas.md,
prefixed with that date and time:

$ARGUMENTS
```

**▶️ `/note Maybe we should cache the homepage`**

**✅** A new, date-stamped line is added to `ideas.md`.

---

# Part 4 — Settings You Can Add (Options)

These go in the **frontmatter** (markdown) or the JSON config. All are optional
except the prompt itself.

| Setting | What it does | Example value |
| --- | --- | --- |
| `description` | Short text shown next to the command | `Draft a commit message` |
| `agent` | Which agent runs the command | `build` or `plan` |
| `subtask` | Runs as a sub-agent to keep main context clean | `true` |
| `model` | Use a specific model for this command | `anthropic/claude-3-5-sonnet-20241022` |

**Frontmatter version**

```markdown
---
description: Draft a commit message
agent: build
subtask: true
model: anthropic/claude-3-5-sonnet-20241022
---

Write a commit message for the staged changes.
```

**JSON version** (in `acoeai.jsonc`)

```jsonc
{
  "$schema": "https://raw.githubusercontent.com/TestyantraSolutions/ACOE/refs/heads/main/config.json",
  "command": {
    "commit": {
      "template": "Write a commit message for the staged changes.",
      "description": "Draft a commit message",
      "subtask": true
    }
  }
}
```

> 💡 **When to use `subtask: true`?** When the command is a side task you don't want
> cluttering your main conversation — like a quick review or a summary.

---

# Part 5 — Where Files Go & How Names Work

| Scope | Folder | Best for |
| --- | --- | --- |
| **Global** | `~/.local/share/acoeai/config/commands/` | Your personal commands, available everywhere |
| **Project** | `.acoeai/commands/` | Team commands shared through git |

**Naming rule:** the file name *is* the command name.

```
readme.md      ->  /readme
todo-scan.md   ->  /todo-scan
fix-spelling.md ->  /fix-spelling
```

> ⚠️ Your custom command can **override a built-in** one. If you create `help.md`,
> then `/help` becomes *your* command instead of the built-in one. So avoid built-in
> names unless you really mean to replace them.

---

# Part 6 — Cheat Sheet

```
PLACE IT HERE
  Global    ~/.local/share/acoeai/config/commands/<name>.md
  Project   .acoeai/commands/<name>.md
  Config    acoeai.jsonc  ->  "command": { "<name>": { ... } }

THE NAME = THE FILE NAME
  note.md  ->  /note

DYNAMIC TOOLS
  $ARGUMENTS        everything typed after the command
  $1  $2  $3        one argument each
  !`date`           live shell output, pasted in
  @ideas.md         contents of a file, pasted in

OPTIONAL FRONTMATTER
  ---
  description: ...
  agent: ...
  subtask: true
  model: ...
  ---

RUN IT
  /name optional arguments here
```

---

# Part 7 — Common Mistakes & Friendly Tips

| Mistake | Easy fix |
| --- | --- |
| Command name doesn't work | The name comes from the **file name**, not the frontmatter |
| Arguments with spaces get split | Wrap them in quotes: `/email "take Friday off"` |
| Shell output looks wrong | Shell commands run from the **project root** — check the path |
| Frontmatter is ignored | Make sure `---` is on the **very first line** of the file |
| Main chat gets messy | Add `subtask: true` to run it in the background |
| Accidentally replaced a built-in | Avoid names like `help`, `init`, `undo` |
| JSON command does nothing | JSON commands **require** the `template` field |

**Remember these five things and you're set:**

1. A command is just a **Markdown file** in a folder.
2. The **file name** is the **command name**.
3. `$ARGUMENTS` and `$1` make it flexible.
4. `!`shell`` and `@file` bring in live context.
5. `subtask: true` keeps your main conversation clean.

Commit your `.acoeai/commands/` folder to your repo, and your whole team gets the
same shortcuts automatically.
