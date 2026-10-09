# AcoeAI Config

## The Idea in One Sentence

> The AcoeAI config is just a **settings file** that tells AcoeAI how you like to work.

Think of it like your phone's settings:

- Some settings apply to **everyone on the device** (global).
- Some apply to **just one app** (project).
- A company can **lock** certain settings so they can't be changed (managed).

That's the whole idea. Everything else is just "which setting does what."

---

## Table of Contents

1. [The Very Basics (Format)](#1-the-very-basics-format)
2. [Where Config Files Live & Who Wins](#2-where-config-files-live--who-wins)
3. [A Guided Tour of Settings](#3-a-guided-tour-of-settings)
4. [Variables — Reusing Env Vars & Files](#4-variables--reusing-env-vars--files)
5. [A Complete Example Config](#5-a-complete-example-config)
6. [Cheat Sheet](#6-cheat-sheet)
7. [Tips & Common Mistakes](#7-tips--common-mistakes)

---

## 1. The Very Basics (Format)

AcoeAI reads a JSON file. If you like keeping notes in your config, use **JSONC**
(JSON with comments) — same thing, but you can write `// comments`.

**📄 `acoeai.jsonc`** — the smallest useful config for Sunflower Notes

```jsonc
{
  "$schema": "https://raw.githubusercontent.com/TestyantraSolutions/ACOE/refs/heads/main/config.json",

  // Which model I want by default
  "model": "anthropic/claude-haiku-4-5",

  // Keep AcoeAI fresh automatically
  "autoupdate": true,
}
```

> 💡 **What each line does**
> - `$schema` — gives your editor autocomplete and error-checking. Always include it.
> - `model` — the AI model AcoeAI uses unless something else overrides it.
> - `autoupdate` — download new versions when AcoeAI starts.

That's a valid config already. The rest of this guide adds more knobs, one at a time.

---

## 2. Where Config Files Live & Who Wins

### The key rule

> **Config files are merged, not replaced.**

Imagine stacking sheets of paper: each new sheet **adds** settings, and if two sheets set
the *same* setting, the **later sheet wins** for that one line only. Everything else
stays.

**Example:**

```text
Global config sets   ->  "autoupdate": true
Project config sets  ->  "model": "anthropic/claude-sonnet-4-5"

Result = BOTH are active.
```

### The full precedence order (last one wins)

| Order | Where it comes from | Think of it as… |
| --- | --- | --- |
| 1 | **Global config** (`~/.local/share/acoeai/config/acoeai.json`) | Your personal preferences |
| 2 | **Custom config** (`ACOEAI_CONFIG` env var) | A one-off override file |
| 3 | **Project config** (`acoeai.json` in the repo) | This project's rules |
| 4 | **`.acoeai` directories** | Project agents, commands, plugins |
| 5 | **Inline config** (`ACOEAI_CONFIG_CONTENT` env var) | Runtime-only tweaks |
| 6 | **Managed files** (e.g. `/etc/acoeai/`) | Admin rules |
| 7 | **macOS managed preferences** | Locked by IT, cannot be overridden |

> 💡 **Simple takeaway:** project beats global, and anything "managed" beats everything.

### Global config — your personal defaults

**📄 `~/.local/share/acoeai/config/acoeai.json`**

```json
{
  "$schema": "https://raw.githubusercontent.com/TestyantraSolutions/ACOE/refs/heads/main/config.json",
  "model": "anthropic/claude-haiku-4-5",
  "share": "disabled"
}
```

- Prefers a **fast, cheap model** for everyday chat.
- Turns sharing **off** for privacy.
- Uses the **gruvbox** colour theme.

### Per-project config — just for one repo

Put `acoeai.json` in the **root of the project**. It wins over global settings.

**📄 `acoeai.json`** (in the Sunflower Notes repo)

```json
{
  "$schema": "https://raw.githubusercontent.com/TestyantraSolutions/ACOE/refs/heads/main/config.json",
  "model": "anthropic/claude-sonnet-4-5",
  "watcher": {
    "ignore": ["coverage/**", "tmp/**", ".cache/**"]
  }
}
```

- The **project uses a smarter model** than the global one.
- The file watcher **skips noisy folders** so things stay fast.

> ✅ Project config is **safe to commit to Git**. When AcoeAI starts, it looks in the
> current folder and walks up to the nearest Git root.

### Custom path — a config from somewhere else

Point AcoeAI at a specific file with an environment variable.

**Terminal**

```bash
export ACOEAI_CONFIG=/home/nessa/sunflower-special.json
acoeai run "summarize my notes"
```

You can also point to a whole **directory** of agents/commands with
`ACOEAI_CONFIG_DIR`.

### Managed settings — rules users can't change

Admins can place a config in a system folder that regular users can't write to:

| Platform | Folder |
| --- | --- |
| macOS | `/Library/Application Support/acoeai/` |
| Linux | `/etc/acoeai/` |
| Windows | `%ProgramData%\acoeai` |

On macOS, IT can also deploy managed preferences through MDM. These take the
**highest priority** — nothing a user does can override them.

> 💡 **If you're not an admin, you can skip this section.** It only matters in companies.

---

## 3. A Guided Tour of Settings

I've grouped the settings by what they're *for*, so it's easier to remember.

---

### 3.1 Models & Providers

Pick your main model, a cheap "small" model, and tune provider behaviour.

```json
{
  "$schema": "https://raw.githubusercontent.com/TestyantraSolutions/ACOE/refs/heads/main/config.json",
  "model": "anthropic/claude-sonnet-4-5",
  "small_model": "anthropic/claude-haiku-4-5",
  "provider": {
    "anthropic": {
      "options": {
        "timeout": 450000,
        "chunkTimeout": 45000
      }
    }
  }
}
```

| Setting | What it does |
| --- | --- |
| `model` | Your main model for real work |
| `small_model` | A cheaper model for small jobs like naming a session |
| `provider.*.options.timeout` | How long to wait for a response (ms) |
| `provider.*.options.chunkTimeout` | How long to wait between streamed chunks |

> 💡 `small_model` is a money-saver: quick tasks don't need the big model.
> Run `acoeai models` to see the available model names.

---

### 3.2 Agents & Tools

**Pick a default agent.** The default must be a *primary* agent (not a subagent):

```json
{
  "$schema": "https://raw.githubusercontent.com/TestyantraSolutions/ACOE/refs/heads/main/config.json",
  "default_agent": "plan"
}
```

**Define a custom agent right in the config.** Here's a read-only note summarizer:

```json
{
  "$schema": "https://raw.githubusercontent.com/TestyantraSolutions/ACOE/refs/heads/main/config.json",
  "agent": {
    "note-summarizer": {
      "description": "Summarizes notes without changing them",
      "model": "anthropic/claude-haiku-4-5",
      "prompt": "Summarize the given notes in 5 bullet points. Never edit files.",
      "tools": {
        "write": false,
        "edit": false
      }
    }
  }
}
```

> You can also define agents as Markdown files in `.acoeai/agents/`.

**Turn tools on or off globally:**

```json
{
  "$schema": "https://raw.githubusercontent.com/TestyantraSolutions/ACOE/refs/heads/main/config.json",
  "tools": {
    "webfetch": false,
    "task": false
  }
}
```

**Limit how deep subagents can go:**

```json
{
  "$schema": "https://raw.githubusercontent.com/TestyantraSolutions/ACOE/refs/heads/main/config.json",
  "subagent_depth": 2
}
```

> `subagent_depth` guide: `0` = no subagents at all, `1` = subagents but they can't
> spawn more (default), `2` = one extra nesting level.

**Add standing instructions** (an array of files or globs):

```json
{
  "$schema": "https://raw.githubusercontent.com/TestyantraSolutions/ACOE/refs/heads/main/config.json",
  "instructions": ["AGENTS.md", "docs/*.md"]
}
```

---

### 3.3 Permissions — staying safe

By default AcoeAI allows everything. To make it **ask first**, or **never**, change
`permission`. Each key is `allow`, `ask`, or `deny`.

```json
{
  "$schema": "https://raw.githubusercontent.com/TestyantraSolutions/ACOE/refs/heads/main/config.json",
  "permission": {
    "edit": "ask",
    "bash": "deny"
  }
}
```

You can also allow only **specific** shell commands:

```json
{
  "$schema": "https://raw.githubusercontent.com/TestyantraSolutions/ACOE/refs/heads/main/config.json",
  "permission": {
    "bash": {
      "*": "ask",
      "git status": "allow",
      "ls": "allow"
    }
  }
}
```

> **Last matching rule wins** — so put the broad `*` rule first and specific rules after.

---

### 3.4 Workflow Boosters

These small settings save a lot of time. Each is optional — add only what you need.

**Run a formatter automatically** — here we use **Biome** instead of Prettier:

```json
{
  "$schema": "https://raw.githubusercontent.com/TestyantraSolutions/ACOE/refs/heads/main/config.json",
  "formatter": {
    "prettier": { "disabled": true },
    "biome": {
      "command": ["npx", "@biomejs/biome", "format", "--write", "$FILE"],
      "extensions": [".js", ".ts", ".json"]
    }
  }
}
```

**Turn on LSP** for real code intelligence (and disable one you don't need):

```json
{
  "$schema": "https://raw.githubusercontent.com/TestyantraSolutions/ACOE/refs/heads/main/config.json",
  "lsp": {
    "ruby-lsp": { "disabled": true }
  }
}
```

**Ignore folders** so the file watcher stays quiet:

```json
{
  "$schema": "https://raw.githubusercontent.com/TestyantraSolutions/ACOE/refs/heads/main/config.json",
  "watcher": {
    "ignore": ["coverage/**", "tmp/**", ".cache/**"]
  }
}
```

**Control context compaction** (keeps long chats from overflowing):

```json
{
  "$schema": "https://raw.githubusercontent.com/TestyantraSolutions/ACOE/refs/heads/main/config.json",
  "compaction": {
    "auto": true,
    "prune": true,
    "reserved": 12000
  }
}
```

**Manage snapshots** (used for undo/redo; disable on huge repos):

```json
{
  "$schema": "https://raw.githubusercontent.com/TestyantraSolutions/ACOE/refs/heads/main/config.json",
  "snapshot": false
}
```

**Autoupdate options:**

```json
{
  "$schema": "https://raw.githubusercontent.com/TestyantraSolutions/ACOE/refs/heads/main/config.json",
  "autoupdate": "notify"
}
```

> `true` = update silently, `"notify"` = just tell me, `false` = do nothing.

**Add plugins** (from npm or local folders):

```json
{
  "$schema": "https://raw.githubusercontent.com/TestyantraSolutions/ACOE/refs/heads/main/config.json",
  "plugin": ["sunflower-notes-helper"]
}
```

**Define custom commands** (shortcut prompts):

```json
{
  "$schema": "https://raw.githubusercontent.com/TestyantraSolutions/ACOE/refs/heads/main/config.json",
  "command": {
    "tidy": {
      "template": "Tidy up the notes in @notes/ and fix any formatting.",
      "description": "Tidy the notes folder"
    }
  }
}
```

**Connect MCP servers** (external tools):

```json
{
  "$schema": "https://raw.githubusercontent.com/TestyantraSolutions/ACOE/refs/heads/main/config.json",
  "mcp": {}
}
```

---

### 3.5 Providers, Policies & Shell

**Allow only certain providers** (an allowlist):

```json
{
  "$schema": "https://raw.githubusercontent.com/TestyantraSolutions/ACOE/refs/heads/main/config.json",
  "enabled_providers": ["anthropic"]
}
```

**Or block specific providers** (a blocklist):

```json
{
  "$schema": "https://raw.githubusercontent.com/TestyantraSolutions/ACOE/refs/heads/main/config.json",
  "disabled_providers": ["openai"]
}
```

> ⚠️ If a provider is in **both** lists, `disabled_providers` wins.

**Policies** let you deny an action on a resource (currently for providers):

```json
{
  "$schema": "https://raw.githubusercontent.com/TestyantraSolutions/ACOE/refs/heads/main/config.json",
  "experimental": {
    "policies": [
      {
        "effect": "deny",
        "action": "provider.use",
        "resource": "azure"
      }
    ]
  }
}
```

**Choose your shell:**

```json
{
  "$schema": "https://raw.githubusercontent.com/TestyantraSolutions/ACOE/refs/heads/main/config.json",
  "shell": "bash"
}
```

> Leave it out and AcoeAI picks a sensible default for your operating system.

> ⚠️ The `experimental` section is unstable — expect changes.

---

## 4. Variables — Reusing Env Vars & Files

Two handy placeholders keep secrets and big files out of your config.

### Env vars — `{env:NAME}`

```json
{
  "$schema": "https://raw.githubusercontent.com/TestyantraSolutions/ACOE/refs/heads/main/config.json",
  "model": "{env:SUNFLOWER_MODEL}",
  "provider": {
    "anthropic": {
      "options": {
        "apiKey": "{env:SUNFLOWER_API_KEY}"
      }
    }
  }
}
```

> If the environment variable isn't set, it becomes an empty string.

### Files — `{file:path}`

```json
{
  "$schema": "https://raw.githubusercontent.com/TestyantraSolutions/ACOE/refs/heads/main/config.json",
  "provider": {
    "anthropic": {
      "options": {
        "apiKey": "{file:~/.secrets/sunflower-key}"
      }
    }
  }
}
```

**Why this is nice:**

- 🔐 Keep API keys in a separate file, out of your main config.
- 📚 Include large instruction files without bloating the config.
- ♻️ Share a common snippet across several config files.

> Paths can be **relative to the config file** or **absolute** (`/...` or `~`).

---

## 5. A Complete Example Config

Here's everything above combined into one believable setup for **Sunflower Notes**.

**📄 `acoeai.json`** (project root)

```jsonc
{
  "$schema": "https://raw.githubusercontent.com/TestyantraSolutions/ACOE/refs/heads/main/config.json",

  // ---- Models ----
  "model": "anthropic/claude-sonnet-4-5",
  "small_model": "anthropic/claude-haiku-4-5",

  // ---- Agent behaviour ----
  "default_agent": "plan",
  "subagent_depth": 1,
  "instructions": ["AGENTS.md", "docs/*.md"],
  "agent": {
    "note-summarizer": {
      "description": "Summarizes notes without changing them",
      "tools": { "write": false, "edit": false }
    }
  },

  // ---- Safety ----
  "permission": {
    "edit": "ask",
    "bash": {
      "*": "ask",
      "git status": "allow"
    }
  },

  // ---- Workflow ----
  "formatter": {
    "prettier": { "disabled": true },
    "biome": {
      "command": ["npx", "@biomejs/biome", "format", "--write", "$FILE"],
      "extensions": [".js", ".ts", ".json"]
    }
  },
  "watcher": {
    "ignore": ["coverage/**", "tmp/**", ".cache/**"]
  },
  "compaction": { "auto": true, "prune": true },

  // ---- Custom command ----
  "command": {
    "tidy": {
      "template": "Tidy up the notes in @notes/ and fix formatting.",
      "description": "Tidy the notes folder"
    }
  }
}
```

A realistic, working setup — and every setting has a reason to be there.

---

## 6. Cheat Sheet

```
FILE FORMAT
  acoeai.json / acoeai.jsonc   (JSON or JSON with comments)
  Always include  "$schema": "https://raw.githubusercontent.com/TestyantraSolutions/ACOE/refs/heads/main/config.json"

WHERE IT LIVES
  Global    ~/.local/share/acoeai/config/acoeai.json
  Project   ./acoeai.json
  Custom    $ACOEAI_CONFIG=/path/file.json
  Dir       $ACOEAI_CONFIG_DIR=/path/dir
  Managed   /etc/acoeai/ (Linux) etc.

WHO WINS (last wins)
  remote < global < custom < project < .acoeai < inline < managed

PLACEHOLDERS
  {env:NAME}          value from an environment variable
  {file:path}         value from a file's contents
```

---

## 7. Tips & Common Mistakes

| Mistake | Easy fix |
| --- | --- |
| Setting seems ignored | Something with **higher precedence** overrides it (project beats global) |
| Comments break the file | Use `.jsonc`, not `.json`, when adding `//` comments |
| `default_agent` doesn't apply | It must be a **primary** agent, not a subagent |
| Provider still loads | Check `disabled_providers` — it beats `enabled_providers` |
| Secret leaked in config | Use `{env:...}` or `{file:...}` instead of pasting keys |
| Shell commands behave oddly | Permission rules use **last-match-wins**; put `*` first |

**Remember these five things and you're set:**

1. Config is just a **settings file** — JSON or JSONC.
2. Files **merge**, and the **later source wins** for conflicts.
3. **Project** config beats **global**; **managed** beats everything.
4. Group settings by purpose: **models, agents, safety, workflow, appearance**.
5. Use `{env:...}` and `{file:...}` to keep secrets out of your config.

Commit your project `acoeai.json` (and `.acoeai/` folder) to Git, and your whole
team gets the same setup automatically.
