# AcoeAI Agents

## The Idea in One Sentence

> An **agent** is a specialized version of the AI assistant with its own instructions,
> its own model, and its own rules about what it's allowed to touch.

**The best way to think about it:** imagine you run a small workshop.

- Your **main helper** can do almost anything (write, edit, run commands).
- But sometimes you want a **specialist** — a proofreader, a teacher, a note-taker —
  who does *one job* and never wanders off.

Agents let you create those specialists.

---

## Table of Contents

1. [Two Kinds of Agents](#1-two-kinds-of-agents)
2. [The Built-in Agents](#2-the-built-in-agents)
3. [How to Use Agents](#3-how-to-use-agents)
4. [Creating Your Own Agent](#4-creating-your-own-agent)
5. [All the Settings (Options)](#5-all-the-settings-options)
6. [Permissions Made Simple](#6-permissions-made-simple)
7. [Original Example Agents](#7-original-example-agents)
8. [Cheat Sheet](#8-cheat-sheet)
9. [Tips & Common Mistakes](#9-tips--common-mistakes)

---

## 1. Two Kinds of Agents

There are only two types. Once you understand this, everything else clicks.

| Type | What it is | How you reach it | Think of it as… |
| --- | --- | --- | --- |
| **Primary** | The main assistant you chat with | Cycle with the **Tab** key | Your main helper |
| **Subagent** | A specialist called in for a task | `@mention` it, or the main agent calls it | A specialist you phone up |

### Primary agents

- You talk to them **directly**.
- Press **Tab** (or your configured `switch_agent` keybind) to switch between them.
- They run the **main conversation**.
- Their power is controlled by **permissions** — e.g. one can edit files, another can't.

### Subagents

- They do a **specific job** (explore, research, review…).
- A primary agent can **call them automatically** for the right task.
- You can also **call them yourself** by typing `@name`:

  ```
  @mentor please explain this file to me
  ```

- Each subagent runs in its **own child session**, so your main chat stays tidy.

> 💡 **Nice trick:** When a subagent opens a child session, press **Down** (Leader + Down)
> to jump into it, **Left/Right** to move between children, and **Up** to return to the
> parent conversation.

---

## 2. The Built-in Agents

AcoeAI ships with these out of the box — no setup needed.

### Primary

| Agent | Mode | What it does (simply) |
| --- | --- | --- |
| **Build** | primary | The **default**. Can do everything — edit files, run commands. Use for real work. |
| **Plan** | primary | A careful thinker. Asks before editing or running commands. Use to plan without changing anything. |

There are also three **hidden system agents** that run on their own and you never pick
them from the menu:

| Hidden agent | Job |
| --- | --- |
| **compaction** | Squashes a long conversation into a short summary |
| **title** | Names your session |
| **summary** | Writes session summaries |

### Subagents

| Agent | Mode | What it does (simply) |
| --- | --- | --- |
| **General** | subagent | A do-it-all researcher for multi-step tasks. Can make changes. Good for running several jobs at once. |
| **Explore** | subagent | A **read-only** code detective. Finds files and answers questions fast. Can't change anything. |
| **Scout** | subagent | A **read-only** researcher for outside code. Clones and inspects dependencies safely. |

> 💡 **Handy rule:** Need to think without breaking things? Use **Plan**. Need to find
> something in a big codebase? Use **Explore**.

---

## 3. How to Use Agents

**Switch primary agents:**

```
Press Tab
```

**Call a subagent by name:**

```
@general help me find every place we use the word "temp"
```

**Jump around subagent sessions:**

| Action | Default key |
| --- | --- |
| Enter the first child session | Leader + Down |
| Next child session | Right |
| Previous child session | Left |
| Back to parent | Up |

---

## 4. Creating Your Own Agent

You can configure built-in agents or make brand-new ones. There are **two ways**, and
both are easy.

### Way 1 — A Markdown file (easiest)

Put a file in one of these folders:

| Scope | Folder |
| --- | --- |
| **Global** (all projects) | `~/.local/share/acoeai/config/agents/` |
| **This project only** | `.acoeai/agents/` |

**The file name becomes the agent name.** `mentor.md` → an agent called `mentor`.

**📄 `.acoeai/agents/mentor.md`**

```markdown
---
description: Explains code in very simple language
mode: subagent
temperature: 0.2
permission:
  edit: deny
  bash: deny
---

You are a patient teacher.
Explain code as if the reader is brand new to programming.
Use short sentences, a small everyday analogy, and finish with one
"why this matters" line.
Never edit files. Only explain.
```

**Use it:**

```
@mentor explain the file src/totals.js
```

### Way 2 — The JSON config

Add an `agent` block to `acoeai.json`.

**📄 `acoeai.json`**

```json
{
  "$schema": "https://raw.githubusercontent.com/TestyantraSolutions/ACOE/refs/heads/main/config.json",
  "agent": {
    "mentor": {
      "description": "Explains code in very simple language",
      "mode": "subagent",
      "temperature": 0.2,
      "prompt": "You are a patient teacher. Explain code simply. Never edit files.",
      "permission": {
        "edit": "deny",
        "bash": "deny"
      }
    }
  }
}
```

> 💡 **Which to choose?** Markdown is nicer for long prompts. JSON is handy if you want
> all your settings in one file.

### Way 3 — Let AcoeAI build it for you

Run this in your terminal:

```
acoeai agent create
```

It asks a few friendly questions (where to save, what the agent should do), generates a
prompt, lets you tick which permissions to allow, and writes the file for you.

---

## 5. All the Settings (Options)

Here's every knob, in plain language.

| Setting | Required? | What it controls |
| --- | --- | --- |
| `description` | **Yes** | A short note about what the agent is for |
| `mode` | No | `primary`, `subagent`, or `all` (default is `all`) |
| `model` | No | Which AI model this agent uses |
| `prompt` | No | The agent's system instructions |
| `temperature` | No | How creative vs. focused it is |
| `top_p` | No | Another way to control randomness |
| `steps` | No | Max number of actions before it must stop and summarize |
| `disable` | No | Set `true` to turn the agent off |
| `hidden` | No | Hide a subagent from the `@` menu |
| `permission` | No | What the agent may do (read, edit, bash, etc.) |
| `color` | No | The agent's color in the UI |

### Description (required)

```json
{
  "agent": {
    "mentor": {
      "description": "Explains code in very simple language"
    }
  }
}
```

### Temperature — creativity dial

Lower = focused and predictable. Higher = creative and varied.

| Range | Best for |
| --- | --- |
| `0.0–0.2` | Explaining, analyzing, careful work |
| `0.3–0.5` | Everyday tasks |
| `0.6–1.0` | Brainstorming, writing, ideas |

Example — a focused explainer and a playful brainstormer:

```json
{
  "agent": {
    "mentor": { "temperature": 0.2 },
    "idea-spark": { "temperature": 0.9 }
  }
}
```

> If you don't set it, AcoeAI uses the model's own default (often `0`, and `0.55` for
> Qwen models).

### Steps — the action limit

Cap how many actions an agent may take. Great for keeping costs down.

```json
{
  "agent": {
    "quick-fix": {
      "description": "Fast help with a small number of steps",
      "prompt": "Solve the problem with as few steps as possible.",
      "steps": 5
    }
  }
}
```

When it hits the limit, it stops and gives you a summary plus suggested next steps.

> ⚠️ The old name `maxSteps` still works but is **deprecated** — use `steps`.

### Disable

```json
{
  "agent": {
    "mentor": { "disable": true }
  }
}
```

### Prompt (point to a file)

```json
{
  "agent": {
    "mentor": { "prompt": "{file:./prompts/teacher.txt}" }
  }
}
```

The path is **relative to the config file**, so it works for both global and project
configs.

### Model

Give a fast model to simple helpers and a stronger model to important work.

```json
{
  "agent": {
    "mentor": { "model": "anthropic/claude-haiku-4-20250514" }
  }
}
```

Format is `provider/model-id`, e.g. `acoeai/gpt-5.1-codex`.

> If you skip this, primary agents use your global model, and subagents inherit the
> model of the primary agent that called them.

### Mode

```json
{
  "agent": {
    "mentor": { "mode": "subagent" }
  }
}
```

Options: `primary`, `subagent`, `all`. Default is `all`.

### Hidden

Hide a helper from the `@` menu so only other agents can call it.

```json
{
  "agent": {
    "quiet-helper": {
      "mode": "subagent",
      "hidden": true
    }
  }
}
```

> Only works for `mode: subagent`. Hidden agents can still be called by other agents.

### Task permissions — who can call whom

Control which subagents an agent may launch.

```json
{
  "agent": {
    "team-lead": {
      "mode": "primary",
      "permission": {
        "task": {
          "*": "deny",
          "helper-*": "allow",
          "mentor": "ask"
        }
      }
    }
  }
}
```

> **Last matching rule wins.** Here `helper-planner` matches both `*` (deny) and
> `helper-*` (allow) — but `helper-*` is later, so the answer is **allow**. And remember:
> you can always `@mention` any subagent yourself, even if task permissions would deny
> it.

### Color

Make agents visually different in the UI.

```json
{
  "agent": {
    "mentor": { "color": "#4ec9b0" },
    "nitpicker": { "color": "warning" }
  }
}
```

Use a hex color like `#4ec9b0`, or a theme name: `primary`, `secondary`, `accent`,
`success`, `warning`, `error`, `info`.

### Top P

An alternative dial to temperature for randomness.

```json
{
  "agent": {
    "idea-spark": { "top_p": 0.9 }
  }
}
```

### Additional (provider-specific extras)

Anything else you add is passed straight through to the model provider. For example,
OpenAI reasoning models support control over reasoning effort:

```json
{
  "agent": {
    "deep-thinker": {
      "description": "Uses high reasoning effort for hard problems",
      "model": "openai/gpt-5",
      "reasoningEffort": "high",
      "textVerbosity": "low"
    }
  }
}
```

> Tip: run `acoeai models` to see the models you can use.

---

## 6. Permissions Made Simple

Permissions decide what an agent is allowed to do. Each one can be:

| Value | Meaning |
| --- | --- |
| `allow` | Do it freely, no asking |
| `ask` | Pause and ask me first |
| `deny` | Never do it |

### The permission keys

| Key | Controls |
| --- | --- |
| `read` | Reading files |
| `edit` | Writing, editing, patching files |
| `glob` | Finding files by pattern |
| `grep` | Searching inside files |
| `list` | Listing directories |
| `bash` | Running terminal commands |
| `task` | Launching subagents |
| `external_directory` | Touching files outside the project |
| `todowrite` | Managing todo lists |
| `webfetch` | Fetching a web page |
| `websearch` | Searching the web |
| `lsp` | Language-server features |
| `skill` | Using skills |
| `question` | Asking questions |
| `doom_loop` | Recovery prompts when an agent gets stuck |

Most of these also accept **wildcard patterns** for fine control.

### Setting a global default, then overriding per agent

```json
{
  "$schema": "https://raw.githubusercontent.com/TestyantraSolutions/ACOE/refs/heads/main/config.json",
  "permission": { "edit": "deny" },
  "agent": {
    "build": {
      "permission": { "edit": "ask" }
    }
  }
}
```

### Allowing only certain shell commands

```json
{
  "agent": {
    "nitpicker": {
      "permission": {
        "bash": {
          "git diff": "allow",
          "git log*": "allow",
          "grep *": "allow",
          "*": "ask"
        }
      }
    }
  }
}
```

> **Order matters — last match wins.** Put the broad `*` rule **first** and specific
> rules **after** it.

### The same thing in a Markdown agent

**📄 `.acoeai/agents/nitpicker.md`**

```markdown
---
description: Finds typos and tiny mistakes, never edits
mode: subagent
permission:
  edit: deny
  bash:
    "*": ask
    "git diff": allow
    "git log*": allow
  webfetch: deny
---

Only look for typos, spelling, and small consistency issues.
Report them as a numbered list. Never change the files.
```

---

## 7. Original Example Agents

Each example is complete and ready to use.

### Example 1 — `mentor` : explains code like a patient teacher

**What it does** Turns confusing code into a friendly explanation. Read-only.

**📄 `.acoeai/agents/mentor.md`**

```markdown
---
description: Explains code in very simple language
mode: subagent
temperature: 0.2
color: success
permission:
  edit: deny
  bash: deny
---

You are a patient teacher.
Explain the given code as if the reader just started learning.
Use short sentences, one everyday analogy, and end with a single
"why this matters" line.
```

**Use:** `@mentor explain src/totals.js`

---

### Example 2 — `nitpicker` : catches typos without touching your files

**What it does** A careful proofreader. Can *read* git info but never edits.

**📄 `.acoeai/agents/nitpicker.md`**

```markdown
---
description: Finds typos and small consistency issues
mode: subagent
permission:
  edit: deny
  bash:
    "git diff": allow
    "git log*": allow
    "*": ask
  webfetch: deny
---

You are a careful proofreader.
Scan for spelling mistakes, inconsistent naming, and stray whitespace.
List each issue as: file -> line -> what's wrong -> suggested fix.
Never edit the files yourself.
```

**Use:** `@nitpicker check my latest changes`

---

### Example 3 — `commit-cop` : enforces your commit message style

**What it does** Reviews a commit message against your rules and suggests a better one.

**📄 `.acoeai/agents/commit-cop.md`**

```markdown
---
description: Checks commit messages follow our style
mode: subagent
permission:
  edit: deny
  bash:
    "git log*": allow
    "*": ask
---

Our commit format is: type(scope): short summary
Valid types: feat, fix, docs, style, refactor, test, chore.

Given a commit message, say whether it follows the format.
If not, rewrite it so it does, and explain the change in one line.
Never edit any files.
```

**Use:** `@commit-cop review "fixed stuff"`

---

### Example 4 — `plain-words` : rewrites technical text for anyone

**What it does** Converts jargon-heavy docs into plain language.

**📄 `.acoeai/agents/plain-words.md`**

```markdown
---
description: Rewrites technical text into plain language
mode: subagent
temperature: 0.3
permission:
  edit: deny
---

You are a translator from "tech speak" to "everyday speak".
Rewrite the text so a smart person with no coding background understands it.
Keep it accurate. Avoid buzzwords. Use short paragraphs.
Do not change any code snippets. Do not edit files.
```

**Use:** `@plain-words rewrite the section in docs/setup.md`

---

### Example 5 — `flashcards` : turns notes into study questions

**What it does** Reads a file and produces question/answer flashcards.

**📄 `.acoeai/agents/flashcards.md`**

```markdown
---
description: Turns a file into study flashcards
mode: subagent
temperature: 0.5
permission:
  edit: deny
  bash: deny
---

Create flashcards from the given file.
Format each one as:
Q: question
A: short answer

Aim for 8-12 cards covering the most important ideas.
Do not edit anything.
```

**Use:** `@flashcards make cards from notes/week1.md`

---

### Example 6 — `scaffolder` : builds new files from a template

**What it does** This one is allowed to **write** files, but **not** run commands.

**📄 `.acoeai/agents/scaffolder.md`**

```markdown
---
description: Creates new files following project conventions
mode: subagent
permission:
  edit: allow
  bash: deny
---

When asked to create a new file, match the existing project style.
Add a short header comment, follow the naming convention, and keep it minimal.
Never run terminal commands.
```

**Use:** `@scaffolder create a new helper called dateUtils`

---

### Example 7 — `note-taker` : keeps a tidy summary log

**What it does** Summarizes long files or discussions into short notes.

**📄 `.acoeai/agents/note-taker.md`**

```markdown
---
description: Summarizes long content into short notes
mode: subagent
temperature: 0.2
permission:
  edit: deny
  bash: deny
---

Summarize the given content in at most 6 bullet points.
Put the most important point first.
Add a final line called "Next step:" with one suggested action.
```

**Use:** `@note-taker summarize meeting-notes.txt`

---

### Example 8 — `idea-spark` : a creative brainstorming buddy

**What it does** A high-creativity agent for naming, ideas, and options.

**📄 `.acoeai/agents/idea-spark.md`**

```markdown
---
description: Generates creative ideas and options
mode: subagent
temperature: 0.9
top_p: 0.9
color: accent
permission:
  edit: deny
  bash: deny
---

You are a friendly brainstorming partner.
Give 10 varied, clearly different ideas on the topic.
Number them and add a one-line note on the trade-off of each.
Be creative but practical.
```

**Use:** `@idea-spark give me names for a caching library`

---

### Example 9 — `team-lead` : an orchestrator that delegates

**What it does** A primary agent allowed to call some helpers but not others.

**📄 `.acoeai/agents/team-lead.md`**

```markdown
---
description: Coordinates work and delegates to helpers
mode: primary
permission:
  task:
    "*": deny
    "helper-*": allow
    mentor: ask
---

You coordinate tasks.
Break work into small pieces and delegate to the right helper.
Always report a short summary when finished.
```

**Use:** switch to it with **Tab**, then ask it to plan a task.

---

## 8. Cheat Sheet

```
TYPES
  primary    -> you chat with it, switch with Tab
  subagent   -> @mention it, or a primary calls it

BUILT-IN
  build, plan                         (primary)
  general, explore, scout             (subagent)
  compaction, title, summary          (hidden)

FILE LOCATION (Markdown)
  Global   ~/.local/share/acoeai/config/agents/<name>.md
  Project  .acoeai/agents/<name>.md
  The FILE NAME = the agent NAME

RUN THE WIZARD
  acoeai agent create

USE ONE
  Press Tab          (primary)
  @name ask it ...   (subagent)

KEY OPTIONS
  description (required), mode, model, prompt, temperature,
  top_p, steps, disable, hidden, permission, color

PERMISSION VALUES
  allow   do it freely
  ask     ask me first
  deny    never

RANDOMNESS
  low temperature  -> focused     (0.0 - 0.2)
  high temperature -> creative    (0.6 - 1.0)
```

---

## 9. Tips & Common Mistakes

| Mistake | Easy fix |
| --- | --- |
| Agent name doesn't show up | The name comes from the **file name**, not frontmatter |
| `description` missing | It's a **required** field — add it |
| Agent edits files when it shouldn't | Add `permission: edit: deny` |
| Shell rules behave oddly | Rules match top-to-bottom; **the last match wins** — put `*` first |
| Subagent clutters the `@` menu | Add `hidden: true` (subagents only) |
| Used `maxSteps` | It's deprecated — use `steps` |
| Wrong model used | Set `model`; otherwise subagents inherit the caller's model |
| Want pure thinking, no changes | Use the built-in **Plan** agent |

**Remember these five things and you're set:**

1. **Primary** = main chat, **Subagent** = specialist you call with `@`.
2. The **file name** becomes the **agent name**.
3. `permission` is how you keep an agent safe (`allow` / `ask` / `deny`).
4. `temperature` low = careful, high = creative.
5. `acoeai agent create` builds one for you interactively.

Keep your `.acoeai/agents/` folder in git, and your whole team shares the same helpful
specialists.
