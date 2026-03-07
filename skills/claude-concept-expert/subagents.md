# Subagents Reference

Expert-level guide to Claude Code subagent configuration and management. Use this to delegate complex tasks, parallelize workflows, and manage permissions precisely.

## Built-in Types

These types require no configuration — reference them directly via `delegate` or `@` syntax.

| Type | Model | Tools | Purpose |
|------|-------|-------|---------|
| Explore | Haiku | Read-only (Read, Glob, Grep, WebFetch, WebSearch) | Fast file discovery, codebase search, documentation review |
| Plan | Inherited | Read-only | Research-heavy planning, no implementation |
| general-purpose | Inherited | All | Complex multi-step tasks, minimal constraints |

## Custom Agents

Define custom agents in `.claude/agents/<agent-name>/AGENT.md` (shared) or `~/.claude/agents/<agent-name>/AGENT.md` (user-level, all projects).

Minimal example:

```yaml
---
name: code-reviewer
description: Reviews code quality, performance, style
tools: [Read, Glob, Grep]
model: sonnet
---

You are an expert code reviewer...
```

Reference as `@code-reviewer` in conversation or via delegate action.

## Frontmatter Fields

| Field | Type | Values | Purpose |
|-------|------|--------|---------|
| `name` | string | unique ID | Used in `@name` references |
| `description` | string | text | When to delegate, what agent does |
| `tools` | list | tool names | Allowlist: only listed tools work |
| `disallowedTools` | list | tool names | Denylist: block specific tools |
| `model` | string | sonnet/opus/haiku/inherit | Inference model; inherit uses parent's |
| `permissionMode` | string | (see table below) | Permission checking strategy |
| `maxTurns` | number | integer | Auto-stop after N conversation turns |
| `skills` | list | skill names | Preload skills (e.g., `documentation-engineer`) |
| `memory` | string | user/project/local | Persistent memory scope |
| `isolation` | string | worktree | Enable git worktree per subagent run |
| `background` | boolean | true/false | Run concurrently with main conversation |
| `hooks` | object | {onStart, onFinish} | Lifecycle triggers |

**Key rules:**
- `tools` and `disallowedTools` are mutually exclusive — prefer `tools` for allowlisting.
- `model: inherit` uses the parent agent's model (useful for skill-based delegation).
- `permissionMode` gates tool execution; pair with appropriate `tools` list.

## Permission Modes

| Mode | Behavior | Use Case |
|------|----------|----------|
| `default` | Prompt user for each new tool | Interactive tasks requiring sign-off |
| `acceptEdits` | Auto-accept Read/Edit/Write/Delete; prompt for others | Trusted editing tasks |
| `dontAsk` | Auto-deny unknown tools; skip prompts | Strict boundary enforcement |
| `bypassPermissions` | Skip all checks (dangerous) | Fully autonomous agents in sandboxed contexts |
| `plan` | Read-only; deny all write tools | Planning and research phases |

**Recommendation:** Pair `acceptEdits` with a tight `tools` list for reliable automation. Example:
```yaml
tools: [Read, Glob, Grep, Edit, Write]
permissionMode: acceptEdits
```

## Worktree Isolation

Set `isolation: worktree` to give each agent run its own git branch and file state.

**Behavior:**
- Agent creates a separate branch per run (auto-named).
- Changes are isolated; no interference with main workspace.
- Shared `.git` history — parent can see agent commits.
- Auto-cleanup if no changes detected.
- Enables true parallelization without file conflicts.

**When to use:**
- Multiple agents modifying the same files in parallel.
- Testing potentially breaking changes safely.
- Experimenting without affecting main branch.

## Foreground vs Background

| Aspect | Foreground | Background |
|--------|-----------|------------|
| Blocking | Yes — pauses main conversation | No — runs concurrently |
| Interaction | Interactive prompts, feedback loops | Non-interactive, pre-approved |
| Use Case | Complex tasks needing mid-flight decisions | Fire-and-forget, autonomous work |
| Control | `Ctrl+B` to background | `Ctrl+F` to foreground (if supported) |

**Best practices:**
- Foreground: code review, problem-solving, decision points.
- Background: CI/CD tasks, parallel batch operations, document generation.
- Set `permissionMode` before backgrounding to avoid permission prompts.

## Persistent Memory

Agents can maintain session-independent memory via `memory: <scope>`.

| Scope | Path | Visibility | Persistence |
|-------|------|-----------|------------|
| `user` | `~/.claude/agent-memory/<name>/` | All projects | Across sessions |
| `project` | `.claude/agent-memory/<name>/` | This project only | Across sessions |
| `local` | `.claude/agent-memory-local/<name>/` | Local only | Not shared |

**Loading:**
- First 200 lines of `MEMORY.md` loaded at agent startup.
- Agent can read/write full MEMORY.md during execution.

**Use cases:**
- Learned patterns (e.g., "this codebase prefers functional over OO").
- Session continuity (e.g., "last failure was...").
- Knowledge base (e.g., "team conventions").

## Agent Placement

| Location | Scope | Edit Mode | Version Control |
|----------|-------|-----------|-----------------|
| `.claude/agents/` (project) | All team members | Committed | Git-tracked |
| `~/.claude/agents/` (user) | Personal | Local | Not shared |
| CLI flag `--agents` JSON | Session only | Inline | Transient |

**Strategy:** Commit stable, team-wide agents to `.claude/agents/`. Use `~/.claude/agents/` for personal workflows.

## Common Mistakes

1. **Nested subagents** — Subagents cannot spawn their own subagents. Design workflows as flat delegation.

2. **Over-permissive tools** — Giving all tools defaults to full access. Allowlist tightly: `tools: [Read, Glob, Grep]` instead of implicit all.

3. **Foreground for long tasks** — Long-running tasks block main conversation. Use `background: true` for autonomous work unless interactive feedback is essential.

4. **Background for decision gates** — Background tasks auto-approve all tools, missing manual review opportunities. Use foreground when approval is needed.

5. **Forgetting `permissionMode`** — Defaults to `default` (interactive prompts on every new tool). Set explicitly: `permissionMode: acceptEdits` for trusted agents.

6. **Memory without structure** — MEMORY.md grows unbounded. Establish a convention: e.g., "Keep to 200 lines; summarize old entries weekly."

7. **Isolation without cleanup** — Worktrees accumulate if branches aren't deleted. Document cleanup responsibility in agent description.

## Quick Start

Create a simple code-review agent:

```yaml
---
name: code-reviewer
description: Synchronous code review for style, performance, types
tools: [Read, Glob, Grep]
model: sonnet
permissionMode: plan
maxTurns: 5
---

You are a senior software engineer. Review code for clarity,
performance, type safety, and alignment with Clean Architecture.
```

Reference: `@code-reviewer review this function signature`.

For complex parallel work, add isolation:

```yaml
---
name: feature-builder
isolation: worktree
background: true
permissionMode: acceptEdits
tools: [Read, Glob, Grep, Edit, Write]
---

You implement features from detailed plans...
```

Delegate with confidence. Subagents are designed for autonomy.
