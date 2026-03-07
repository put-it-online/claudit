---
name: claude-concept-expert
description: Use when working with Claude Code configuration - CLAUDE.md files, skills, hooks, rules, subagents, slash commands, context window optimization, or workflows combining these concepts. Triggers on questions about or modifications to .claude/ directory contents, CLAUDE.md structuring, skill authoring, hook configuration, or context management.
user-invocable: false
---

# Claude Concept Expert

Expert knowledge for Claude Code configuration and workflow systems. For experienced users optimizing their setup or building new configurations.

## When This Skill Applies

- Creating, editing, or auditing CLAUDE.md files
- Authoring or debugging skills (SKILL.md, frontmatter, supplementary files)
- Configuring hooks (events, matchers, types)
- Organizing .claude/rules/ files
- Creating or configuring subagents
- Optimizing context window usage
- Setting up a complete Claude Code workflow

## How to Use This Skill

**Identify which domain(s) the request touches, then read the relevant reference file(s) before responding.**

### Domain Reference Map

| User intent | Read this file |
|-------------|---------------|
| CLAUDE.md structure, hierarchy, what to include/exclude, memory | [claude-md.md](claude-md.md) |
| Creating skills, SKILL.md format, frontmatter, discovery, placement | [skills-authoring.md](skills-authoring.md) |
| Hook configuration, events, matchers, patterns | [hooks.md](hooks.md) |
| .claude/rules/ organization, path-specific rules | [rules.md](rules.md) |
| Subagent types, custom agents, isolation, permissions | [subagents.md](subagents.md) |
| Context window size, compaction, token budgets, optimization | [context-window.md](context-window.md) |
| Combining concepts, complete setup, audit checklist | [workflows.md](workflows.md) |

**Multiple domains:** If the request spans multiple areas (e.g., "set up my project from scratch"), read [workflows.md](workflows.md) first, then load domain-specific files as needed.

## Behavior Modes

**Advisory** — When the user asks questions ("How should I structure my CLAUDE.md?"):
1. Load relevant reference file(s)
2. Provide specific, actionable guidance
3. Include anti-patterns to avoid
4. Reference concrete examples

**Builder** — When the user requests action ("Create a hook that auto-formats after edits"):
1. Load relevant reference file(s)
2. Create or edit the actual files
3. Explain key decisions made
4. Note any configuration the user needs to complete

## Concept Overview (for routing — do NOT use as sole source)

These summaries help identify which file to load. Always read the full reference before advising.

- **CLAUDE.md**: Project memory file loaded every session. 3-level hierarchy (global/project/subdirectory). Hard 200-line limit. Use progressive disclosure.
- **Skills**: Folders with SKILL.md + optional supplementary files. Auto-triggered or manual. 4 placement levels. Progressive loading (metadata at startup, full on-demand).
- **Hooks**: Shell commands/prompts/agents that fire at lifecycle events. 16+ event types. Control via exit codes and JSON. Matchers filter by tool/event.
- **Rules**: Modular `.claude/rules/*.md` files. Auto-discovered. Support path-specific activation via frontmatter. Lower token cost than CLAUDE.md.
- **Subagents**: Isolated context windows for delegated work. Built-in types (Explore, Plan, general-purpose) + custom. Worktree isolation for parallel work.
- **Context Window**: 200K total, ~130K usable after overhead. Compaction at ~95%. Performance degrades at ~147K. Progressive disclosure is the key optimization.
- **Workflows**: How all concepts compose. CLAUDE.md + rules + skills + hooks form a layered system. Three loading tiers: always, on-demand, invoked.

## Important

- This skill targets **experienced** Claude Code users. Don't over-explain basics.
- Always load reference files before responding — the overviews above are insufficient for expert guidance.
- When building, follow the patterns in the reference files exactly (frontmatter format, file locations, naming conventions).
- If the user's request spans the full setup, start with [workflows.md](workflows.md) for the big picture.
