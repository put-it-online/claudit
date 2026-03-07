# Skills Authoring Reference

Expert-level guidance for building effective Claude Code skills. Target: 600–800 words, reference-style format.

## Directory Structure

```
skill-name/
├── SKILL.md              # Required: skill definition
├── references/           # Optional: >100 line docs
│   ├── patterns.md
│   └── troubleshooting.md
├── scripts/              # Optional: executable utilities
│   └── validate.sh
└── assets/               # Optional: templates, examples
    └── template.md
```

**Rules:**
- SKILL.md is entry point — keep under 500 lines
- Extract supplementary content when >100 lines or heavily reused
- Reference from SKILL.md with context on WHEN to read
- Claude loads on-demand; do NOT use `@path` links (burns context)

---

## Frontmatter Reference Table

| Field | Type | Default | Purpose |
|-------|------|---------|---------|
| `name` | string | directory name | Skill identifier: lowercase, hyphens, max 64 chars |
| `description` | string | — | **Critical for discovery.** When to auto-invoke. Max 1024 chars. |
| `user-invocable` | boolean | true | Show skill in `/` command menu |
| `disable-model-invocation` | boolean | false | Prevent Claude auto-invoking this skill |
| `allowed-tools` | string list | — | Auto-approved tools: `Read`, `Grep`, `Bash(git *)` |
| `model` | string | inherited | Override model: `sonnet`/`opus`/`haiku`/`inherit` |
| `context` | string | — | Set to `fork` for isolated subagent context |
| `agent` | string | general-purpose | Agent type when `context: fork` |
| `argument-hint` | string | — | CLI autocomplete hint: `[issue-number]` or `[file-path]` |
| `hooks` | object | — | Lifecycle hooks: `on-invoke`, `on-complete`, `on-error` |

---

## Description Writing (Critical for Skill Optimization)

The description is **injected into Claude's system prompt** when deciding whether to auto-invoke.

**Do:**
- Start with "Use when..." — be explicit about triggering conditions
- List symptoms, situations, contexts where skill applies
- Keep under 500 characters (descriptions >1KB get truncated)
- Write in third person (you → the user, Claude → the assistant)

**Don't:**
- Summarize the full workflow — Claude shortcuts it and skips SKILL.md body
- Assume Claude read SKILL.md before deciding to invoke
- Include implementation details or tool lists

**Example:**

```markdown
description: |
  Use when the user asks to review pull request code for architectural
  violations (dependency flow, layer crossing, naming patterns). Applies
  to files in business/, services/, or applications/ directories. Not
  for linting or formatting — only architecture rules.
```

---

## Invocation Control Matrix

| Config | User `/` | Claude auto | Best for |
|--------|----------|-------------|----------|
| Default (`user-invocable: true`, no disable flag) | ✓ Yes | ✓ Yes | General knowledge, helpers |
| `disable-model-invocation: true` | ✓ Yes | ✗ No | Side-effect workflows (/deploy, /commit, /release) |
| `user-invocable: false` | ✗ No | ✓ Yes | Passive knowledge, formatting helpers |
| Both flags set | ✗ No | ✗ No | **Avoid** — unusable skill |

**Guidance:**
- **Side-effect skills**: Always set `disable-model-invocation: true` (/deploy, /commit, schema changes)
- **Knowledge skills**: Leave both flags default for maximum discoverability
- **Background helpers**: Set `user-invocable: false` only if skill clutters user menu

---

## Tool Allowlisting: `allowed-tools`

Speeds up skill invocation by pre-approving tools.

```yaml
allowed-tools:
  - Read
  - Grep
  - Bash(git *)  # Restrict to git-only bash commands
```

**Approved strings:**
- `Read`, `Write`, `Edit`, `Glob`, `Grep` — file operations
- `Bash(git *)` — git-specific bash only
- `Bash(pnpm *)` — pnpm commands
- Full tool names without restrictions: `WebSearch`, `WebFetch`

Do NOT allowlist Bash without restrictions in production skills.

---

## Advanced Patterns

### Isolated Subagent Context

```yaml
context: fork
agent: architecture-reviewer
```

Creates independent context: isolated message history, separate token budget, no bleed-through from user's project state. Use for:
- Specialized reviewers (architecture, security, performance)
- Long-running analysis (multi-file audits)
- Tools requiring isolated state (custom integrations)

### Argument Substitution

```yaml
argument-hint: "[file-path]"
```

In SKILL.md body, reference arguments:
- `$ARGUMENTS` — entire argument string
- `$0`, `$1`, ... — positional args (zero-indexed)
- `${CLAUDE_SKILL_DIR}` — absolute path to skill directory

### Dynamic Preprocessing

Execute shell commands before Claude sees content:

```markdown
!`cat ${CLAUDE_SKILL_DIR}/assets/checklist.md`
```

Runs at skill parse time. Output replaces the command line.

---

## Placement Hierarchy (Priority Order)

| Level | Path | Scope | Use case |
|-------|------|-------|----------|
| Enterprise | Managed settings | Org-wide | Policy enforcers, risk mitigation |
| Project | `.claude/skills/` | Committed to git | Team standards (architecture, testing) |
| Personal | `~/.claude/skills/` | All projects | Personal workflows, helpers |
| Plugin | `plugin-name:skill-name` | Within plugin | Scoped to plugin ecosystem |

**Strategy:** Start in `.claude/skills/` (project-level). Move to personal if universally useful. Namespace plugin skills to avoid collisions.

---

## Common Mistakes & Fixes

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Description summarizes workflow | Claude invokes but ignores body | Start with "Use when..." only; lean on context clues |
| Using `@path` links in SKILL.md | Loads files at startup, wastes context | Reference with markdown links; Claude loads on-demand |
| SKILL.md > 500 lines | Claude truncates mid-read | Extract to `references/` directory |
| Missing `disable-model-invocation` on `/deploy` | Auto-invokes unintended, runs risky actions | Add flag to all side-effect skills |
| Not testing skill with `context: fork` | Works locally, fails in subagent | Always invoke through fork before deploying |

---

## Testing & Validation

Before deploying:

1. **Invoke from CLI**: `claude skill-name [args]`
2. **Check auto-invocation**: Ask Claude a question matching description
3. **Validate tools**: Confirm `allowed-tools` work without approval prompts
4. **Test with fork**: If using `context: fork`, invoke subagent and confirm isolation
5. **Verify context limits**: Estimate token usage; compress if >50% of budget

---

## File Size Guidelines

| File | Limit | Guidance |
|------|-------|----------|
| SKILL.md | 500 lines | Keep lean; move >100-line sections to references/ |
| Single reference file | 300 lines | Split into multiple docs if grows larger |
| Frontmatter | 50 lines | One frontmatter per skill; reuse via inheritance |
| Scripts | 100 lines per script | Executable; document in-script with comments |

