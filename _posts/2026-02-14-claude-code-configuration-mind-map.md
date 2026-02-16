---
title: "The Claude Code Configuration Mind Map: A Complete Mental Model"
date: 2026-02-14
tags: [claude-code, ai-tools, developer-workflow]
excerpt_text: "Claude Code has 6 configuration layers. Here's a mental model that maps every feature to the right question."
---

> **Last verified:** February 14, 2026. Claude Code is actively evolving — some details may have changed since this was written.

Claude Code is powerful but its configuration surface is sprawling — memory files, permissions, MCP servers, skills, hooks, output styles. When you need to configure something, it's hard to know *where* to reach.

This post organizes the entire configuration system into 6 layers, each answering one question. Once you internalize the questions, you'll always know where to go.

---

## The 6 Layers

```
1. "What should Claude know?"         → Memory Layer
2. "What is Claude allowed to do?"    → Permissions Layer
3. "What tools does Claude have?"     → Extension Layer
4. "What can I tell Claude to do?"    → Skills & Agents Layer
5. "What happens automatically?"      → Hooks Layer
6. "How should Claude look and feel?" → UX Layer
```

And one meta-principle that applies to all of them: **The Cascade** — every layer follows the same scope hierarchy.

---

## 1. Memory Layer — "What should Claude know?"

This is where you tell Claude about your project, your conventions, and your preferences.

**Team-shared (in git, everyone gets this):**
- `CLAUDE.md` — project root, the primary instruction file
- `.claude/rules/*.md` — modular, topic-specific rules (e.g., `frontend.md`, `testing.md`)

**Personal (not in git, only you):**
- `CLAUDE.local.md` — your overrides for this specific project
- `~/.claude/CLAUDE.md` — your rules for ALL projects, everywhere
- `MEMORY.md` — Claude's own auto-notes (per-project)

**Enterprise:**
- `/etc/claude-code/CLAUDE.md` — org-wide policy

**Loading order (highest to lowest priority):**

1. Managed policy (`/etc/claude-code/CLAUDE.md`)
2. Project `CLAUDE.md`
3. Project `CLAUDE.local.md`
4. Project `.claude/rules/*.md`
5. User `~/.claude/CLAUDE.md`
6. Auto memory `MEMORY.md`

**When to reach for this:** "Claude keeps doing X wrong" or "our team follows convention Y."

---

## 2. Permissions Layer — "What is Claude allowed to do?"

Controls what Claude can do without asking, what requires confirmation, and what's completely blocked.

**Settings in `settings.json` (any scope):**
- `allow: ["Bash(npm run *)"]` — auto-approve matching commands
- `ask: ["Bash(git push *)"]` — always confirm first
- `deny: ["Read(.env)"]` — hard block, no exceptions

**Pattern syntax:**
- `Bash(npm run *)` — prefix match on commands
- `Read(.env)` — exact file match
- `WebFetch(domain:example.com)` — domain-level control
- `MCP(github)` — MCP server access
- `Task(Explore)` — subagent type control
- `Skill(debug)` — specific skill access

**Sandbox mode:**
- OS-level filesystem isolation
- Network allowlists (domains, unix sockets)
- `excludedCommands` for sandbox bypass
- `autoAllowBashIfSandboxed` — trust commands inside sandbox

**Hooks integration:**
- A `PreToolUse` hook returning exit code 2 blocks the action programmatically

**When to reach for this:** "Claude shouldn't do X" or "auto-approve Y so I stop clicking allow."

---

## 3. Extension Layer — "What tools does Claude have?"

This is how you give Claude new capabilities — connecting it to Slack, Jira, databases, APIs, or anything else.

**MCP Servers — three scopes:**
- `local` (default) — project-specific, private
- `project` (`.mcp.json`, in git) — shared with team
- `user` (`~/.claude.json`) — available in all projects

**Transports:** stdio, http, sse

**Auth:** headers, OAuth (`--client-id`/`--client-secret`)

**Smart scaling:** When MCP tool descriptions exceed 10% of context, `MCPSearch` auto-defers them — only loads tool details when actually needed.

**Management:**
```
claude mcp add/remove/list/get
claude mcp add-from-claude-desktop    # import existing
```

**CLI flags:**
- `--mcp-config custom.json` — one-off tool injection
- `--strict-mcp-config` — ONLY use specified MCP servers
- `--chrome` — enable the browser tool

**When to reach for this:** "I wish Claude could interact with X" — add an MCP server for it.

---

## 4. Skills & Agents Layer — "What can I tell Claude to do on command?"

Reusable workflows you invoke with slash commands, and specialized personas for different tasks.

**Skills (`/slash-commands`):**
- `.claude/skills/<name>/SKILL.md` — project-level
- `~/.claude/skills/<name>/SKILL.md` — personal, all projects

**Skill frontmatter options:**
- `allowed-tools` — restrict which tools the skill can use
- `model` — override which model runs the skill
- `user-only` — only you can invoke it
- `disable-model-invocation` — Claude can't auto-invoke
- `hooks` — skill-scoped automation

**Dynamic context in skills:**
- Backtick commands: `` !`git log --oneline -10` `` — injects live data

**Arguments:** `$ARGUMENTS`, `$0`, `$1`, `$ARGUMENTS[N]`

**Subagents (`.claude/agents/<name>.md`):**
- Scoped tools and model per agent
- Memory frontmatter (scopes: user, project, local)
- Can restrict which subagents it spawns

**Agent Teams (experimental):**
- `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`
- Multi-agent collaboration with task restriction

**When to reach for this:** "I repeat the same workflow often" or "I want a specialized persona."

---

## 5. Hooks Layer — "What happens automatically?"

Event-driven automation that fires before, during, or after Claude's actions.

**Lifecycle events:**
- `Setup` — `--init`, `--init-only`, `--maintenance`
- `SessionStart` — session begins, resumes, or compacts
- `SessionEnd` — session terminates

**User input events:**
- `UserPromptSubmit` — before processing (transform or augment input)

**Tool events:**
- `PreToolUse` — validate or block, inject `additionalContext`
- `PostToolUse` — auto-format, lint after edit
- `PostToolUseFailure` — react to failures
- `PermissionRequest` — permission dialog appears

**Agent events:**
- `SubagentStart` / `SubagentStop` — subagent lifecycle
- `TeammateIdle` — agent team member waiting
- `TaskCompleted` — task marked complete

**Hook types:**
- Shell script (stdin receives JSON context)
- `prompt` — single-turn LLM decision
- `agent` — multi-turn LLM with tools

**Control flow:**
- Exit 0 → allow (stdout injected for some events)
- Exit 2 → block (stderr becomes feedback to Claude)
- JSON stdout → structured control

**Scopes:** global, project, local, plugin, skill, managed

**When to reach for this:** "Every time Claude does X, I want Y to happen automatically."

---

## 6. UX Layer — "How should Claude look and feel?"

Presentation, interaction style, and session management.

**Output styles:**
- `~/.claude/output-styles/<name>.md` — personal
- `.claude/output-styles/<name>.md` — project
- Built-in: `default`, `explanatory`, `learning`
- Switch with `/output-style`

**Keybindings:**
- `~/.claude/keybindings.json`
- 17+ contexts (Chat, Global, Settings, etc.)

**Theme settings:**
- `statusLine` — custom context display (supports `context_window.used_percentage`)
- `spinnerVerbs` — customize spinner text
- `prefersReducedMotion` — accessibility
- `spinnerTipsEnabled` — tips during loading

**Session management:**
- `--continue` / `--resume` — pick up where you left off
- `--from-pr <number>` — resume session linked to a PR
- `--fork-session` — branch off a conversation
- `/rename` — custom session title

**CLI flags:**
- `--verbose` — detailed output
- `--output-format` — text, json, stream-json
- `--effort low|medium|high` — thinking depth

---

## The Cascade — The Meta-Principle

Every layer follows the same scope hierarchy:

```
Managed (/etc/claude-code/)
    ↓ overrides
Enterprise policy
    ↓ overrides
Project (.claude/)
    ↓ overrides
Local (.claude/*.local.*)
    ↓ overrides
User (~/.claude/)
    ↓ overrides
CLI flags (--model, --tools, etc.)
```

This applies to: settings, permissions, MCP, hooks, skills, agents, memory. Same pattern everywhere. Once you understand the cascade for one layer, you understand it for all of them.

---

## The Intuition Cheat Sheet

When you're thinking... reach for this:

- **"Claude keeps doing X wrong"** → Memory layer (`CLAUDE.md`, `.claude/rules/`)
- **"Claude shouldn't do X"** or **"auto-allow Y"** → Permissions (`settings.json`)
- **"I wish Claude could talk to X service"** → MCP server
- **"I do this workflow repeatedly"** → Skill (`.claude/skills/`)
- **"I want a specialized persona"** → Subagent (`.claude/agents/`)
- **"Every time Claude does X, Y should happen"** → Hook
- **"Run setup tasks on new clones"** → Hook (`Setup`) + `--init`
- **"Responses too verbose/short"** → Output style
- **"This rule only applies to frontend"** → `.claude/rules/frontend.md`
- **"Need parallel specialized workers"** → Agent Teams (experimental)
- **"Resume where I left off on a PR"** → `--from-pr` or `--resume`
- **"Limit spend on CI runs"** → `--max-budget-usd`

The configuration surface is large, but the mental model is simple: **6 questions, one cascade**. Match your need to the right question, and you'll always know where to configure it.
