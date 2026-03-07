# Workflows Reference

How all Claude Code concepts compose into deterministic, maintainable workflows.

## How All Concepts Compose

Claude Code loads context in a layered, on-demand architecture:

```
CLAUDE.md (persistent context — every session)
    ↓
.claude/rules/ (conditional context — domain-specific)
    ↓
Skills (auto-discovered capabilities — on relevance)
    ↓
Subagents (delegated execution — isolated context)
    ↓
Hooks (deterministic automation — lifecycle events)
```

**Flow:**
1. Session starts → CLAUDE.md always loaded
2. User mentions a domain → path-matched rules auto-loaded
3. User invokes a repeatable workflow → skill description triggers discovery
4. User delegates work → subagent context narrowed, tools scoped
5. Automated events occur → hooks fire based on matchers

---

## Three-Layer Loading Architecture

| Layer | What Loads | When | Cost |
|-------|-----------|------|------|
| **Always** | CLAUDE.md (root), global rules, skill metadata | Session start | ~20-50K tokens (trim aggressively) |
| **On-demand** | Subdirectory CLAUDE.md, path-specific rules, full SKILL.md | User mentions path or skill | ~5-15K per activation |
| **Invoked** | Skill supplementary files, subagent context, hook scripts | Skill used, subagent delegated, hook fires | ~2-5K per invocation |

**Implication:** Heavy reference files belong in supplementary locations (skill-name/reference.md, not CLAUDE.md).

---

## Audit Checklist

Review your existing setup:

- [ ] CLAUDE.md under 150 lines? No silent truncation?
- [ ] Domain-specific rules moved to `.claude/rules/` instead of inline?
- [ ] Path-specific rules using `paths:` frontmatter where applicable?
- [ ] Skill descriptions trigger-focused (not workflow summaries)?
- [ ] Skills using supplementary files for heavy reference (>100 lines)?
- [ ] Hooks using matchers (not firing on every event)?
- [ ] Context window overhead acceptable? Run `/context` to verify.
- [ ] MCP servers minimal? Each costs 5-10K tokens at session start.
- [ ] Subagents scoped with minimal tool access?
- [ ] No redundant information across CLAUDE.md, rules, and skills?

**Red flags:**
- CLAUDE.md growing past 200 lines → move domain sections to `.claude/rules/`
- Same rule repeated in CLAUDE.md and a `.claude/rules/` file → delete duplicate
- Skill file with heavy reference material → extract to supplementary file
- Hook firing on unrelated events → add matcher to narrow scope

---

## Common Workflow Patterns

### Auto-format Pipeline

**Setup:**
- PostToolUse hook, matcher `Edit|Write`
- Runs prettier/eslint on changed file
- Deterministic — always fires after edits

**Effect:** No need to ask "format this" manually. Every file edit is auto-formatted.

**Example matcher:** `^/(Edit|Write)` fires only after Edit or Write tool calls.

### Context Recovery After Compaction

**Setup:**
- SessionStart hook, matcher `compact`
- Echo critical reminders (current sprint, key conventions, active branch)
- Supplement: CLAUDE.md is re-read automatically, but conversation context is summarized

**Effect:** After compaction, user doesn't need to repeat "remember we're on feature/X" — hook reminds you.

### Skill-Driven Development Workflow

**Sequence:**
1. `brainstorming` skill — explore ideas, scope work
2. `writing-plans` skill — create detailed task specifications
3. `executing-plans` or `subagent-driven-development` skills — implement
4. `finishing-a-development-branch` skill — test, commit, PR

**Enforcement:** Each skill declares REQUIRED SUB-SKILL in metadata, forcing progression.

**Benefit:** Prevents jumping to code without a plan. Catches scope creep early.

### Subagent Delegation

**Use Explore subagent** (read-only, Haiku, fast):
- Audit codebase (grep patterns, file counts)
- Analyze structure, identify entry points
- Extract facts for human-driven planning

**Use custom agents** (scoped tools, specialized role):
- Code review → read-only + lint tools
- Bug debugging → full tools + test runner
- Refactoring → Edit + Glob + Git tools

**Use worktree isolation** (parallel feature work):
- Agent works in `.worktrees/<name>` branch
- Main workspace untouched
- Merge only when ready

**Use background execution** (fire-and-forget):
- Agent runs in background, uploads results
- Human continues work without blocking

---

## Setup from Scratch

**Progressive enhancement:** Start minimal, add complexity only when needed.

### Phase 1: Basics

Create `CLAUDE.md` with:
- Project overview (1-2 sentences)
- Key directory structure (architecture layers)
- Most common commands (test, build, deploy)
- Design constraints (what matters most)
- Size limit: < 80 lines

**Why:** Trimmed CLAUDE.md loads fast (15-20K tokens). Saves 30-50K tokens per session vs. over-documented baseline.

### Phase 2: Modularity

- Move domain-specific context to `.claude/rules/`
- One rule file per domain (architecture, testing, deployment, etc.)
- Add `paths:` frontmatter for path-specific activation

**Why:** Rules load only when relevant. Keeps session context tight.

### Phase 3: Automation

- Add PostToolUse hook for auto-formatting (prettier/eslint)
- Add SessionStart hook for context recovery after compaction
- Test hooks before deploying (run `/init` to see what fires)

**Why:** Removes repetitive manual steps. Hooks are deterministic — always the same output for the same event.

### Phase 4: Skills

- Create project-specific skills for repeatable workflows
- One skill per workflow (e.g., `create-feature`, `debug-test-failure`)
- Personal skills at `~/.claude/skills/` for patterns that apply across projects

**Why:** Skills are discoverable. User types "I need to debug a test" → skill shows up automatically.

### Phase 5: Delegation

- Create custom subagents for specialized roles
- Define tool scope, working directory, context boundaries
- Use worktree isolation for parallel development

**Why:** Subagents execute faster (narrower context), work in parallel, never conflict.

---

## Common Mistakes

**Over-engineering on day 1**
- Start with Phase 1 (CLAUDE.md only).
- Add skills only after you've done the same workflow 3+ times.
- Hooks come last — prove the need first.

**Duplicating information**
- Rule in CLAUDE.md + identical rule in `.claude/rules/` = wasted tokens on every session.
- Solution: Audit for overlap, delete one copy.

**Not testing skills before deploying**
- Skill with typo in trigger → never invoked.
- Skill with incorrect tool list → fails at runtime.
- Solution: Run `/help skill-name` to verify, test with `/use skill-name` before shipping.

**Ignoring context window budget**
- CLAUDE.md 300 lines + 5 MCP servers + 10 rules → 80-100K tokens before your first prompt.
- Solution: Check `/context` after each major addition. Trim aggressively.

**Using hooks for things linters should handle**
- Hook that runs prettier on every write → fights with your editor's auto-format.
- Solution: Let linters handle formatting. Hooks should orchestrate, not duplicate tooling.

---

## Quick Decisions

**When to create a rule:**
- Information that applies to a whole domain (testing, deployment, architecture) → `.claude/rules/`
- Information that applies only within a directory (rules for `/applications/api/`) → path-specific rule with `paths:` frontmatter

**When to create a skill:**
- Workflow you execute 3+ times a month → skill
- Workflow that requires exact ordering (plan → implement → test) → skill with sub-skill markers

**When to create a subagent:**
- Task with tool scope much narrower than your current session (e.g., read-only audit) → subagent
- Task that benefits from parallelization (4+ independent features) → subagent with worktree

**When to create a hook:**
- Event that always requires the same response (auto-format after edit) → hook with matcher
- Event that sometimes needs different responses → manual command instead

---

## Context Window Budget

| Component | Tokens | Trim Strategy |
|-----------|--------|---------------|
| CLAUDE.md | 15-50K | Keep < 80 lines, move rules to `.claude/rules/` |
| One `.claude/rules/` file | 5-15K | Use only when activated (path or domain matcher) |
| One MCP server | 5-10K | Disable unused servers, combine related functionality |
| One skill metadata | 1-2K | Descriptions are auto-loaded; full content only on `/use` |
| Skill supplementary file | 2-10K | Loaded only when skill invoked |
| Subagent context | Variable | Narrow tool list, inherit only essential CLAUDE.md sections |

**Audit:**
```bash
/context  # Shows current breakdown
```

If total > 100K tokens before your input, identify the largest component and trim.

---

## Validation Workflow

Before deploying a new rule/skill/hook:

1. Check syntax: `/init` or `/help component-name`
2. Verify activation: Run the trigger manually, confirm fires
3. Check output: Read hook results or skill metadata — any errors?
4. Audit context: `/context` — is overhead acceptable?
5. Test integration: Does this conflict with existing rules/hooks?

**No manual testing = broken workflows in production.**
