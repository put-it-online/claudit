# Rules Reference

`.claude/rules/` files provide modular, domain-specific guidance organized by concern rather than monolithic instruction. They auto-discover recursively and support conditional activation by file path.

## What Rules Are

Rules are domain-focused markdown files stored in `.claude/rules/` (project-level) or `~/.claude/rules/` (user-level). Each file targets one concern: architecture, testing, security, or domain-specific patterns. Claude loads them conditionally at startup and when matching file paths.

**Key differences from CLAUDE.md:**
- Rules are loaded only when relevant (lower token cost)
- Each file focuses on one domain (higher discoverability)
- Path-specific rules activate only for matching files
- Easier to audit, maintain, and override at different levels

## Directory Structure

Rules organize recursively. No required structure—self-descriptive filenames matter:

```
.claude/rules/
├── architecture.md           # Clean Architecture layers, dependency rules
├── testing.md                # Vitest conventions, mocking patterns
├── endpoints.md              # API endpoint system, URL management
├── changesets-and-deps.md    # Versioning, dependency update policy
├── schemas-and-entities.md   # Entity-as-source-of-truth, Zod workflow
└── frontend/
    ├── react-patterns.md     # Component design, hooks, styling
    └── testing.tsx           # Component testing specifics
```

**Naming guidelines:**
- Use descriptive names (`testing.md`, not `rules1.md`)
- Keep files under ~100 lines (split if exceeding)
- Use `*.md` extension for all rules

## Path-Specific Activation

Rules with `paths:` frontmatter activate only when Claude reads matching files:

```yaml
---
paths:
  - "src/api/**/*.ts"
  - "applications/**/handlers.ts"
---
# API Endpoint Rules

Use the endpoint system, never hardcoded URLs...
```

**Glob patterns supported:**
- `**/*.ts` — all TypeScript files recursively
- `*.md` — markdown files in current directory
- `src/**/*.{ts,tsx}` — TypeScript/TSX in `src/`
- `applications/*/types.ts` — types.ts in any application

Rules **without** `paths:` load unconditionally at startup. Use unconditional rules for general conventions; use path-specific rules for file-type or directory-specific guidance.

## Loading Order

1. **User-level** (`~/.claude/rules/*.md` recursively) — loads first, provides baseline
2. **Project-level** (`.claude/rules/*.md` recursively) — loads second, overrides user-level for conflicts
3. **Unconditional rules** — all loaded at startup
4. **Path-matched rules** — loaded when Claude reads files matching `paths:` glob

This order ensures project-level rules take precedence and path-specific rules apply only when relevant.

## Rules vs CLAUDE.md vs Skills

| Aspect | Rules | CLAUDE.md | Skills |
|--------|-------|-----------|--------|
| Loading | Conditional (path/time) | Every session | On invocation |
| Scope | File/domain-specific | Entire project | Specific workflow |
| Token cost | Lower (selective) | Higher (always) | Lower (on-demand) |
| Reusability | Within project | Via git clone | Across projects |
| Best for | Domain instructions | General conventions | Repeatable tasks |

**When to use each:**
- **CLAUDE.md**: Architecture overview, design constraints, general conventions
- **Rules**: Domain-specific patterns, file-type guidance, conditional workflows
- **Skills**: Multi-step processes, tool orchestration, repeatable external tasks

## Common Mistakes

**Contradicting rules across files** — Multiple rules covering the same topic with different guidance breaks decision-making. Audit for overlaps when adding rules; consolidate when conflicts appear.

**Rules exceeding ~100 lines** — Large rules hoard context. If a file grows beyond 100 lines, split into smaller, focused files: `testing.md`, `testing-patterns.md`, `testing-fixtures.md`.

**Dumping everything into rules** — General project conventions, design philosophy, and architectural overview belong in CLAUDE.md. Reserve rules for specific guidance that doesn't apply everywhere.

**No path-specific activation** — Loading rules for every file (no `paths:` frontmatter) wastes context. Use unconditional rules only for truly universal guidance; apply `paths:` to domain-specific rules.

**Not checking user-level rules** — Project-level rules override user-level. Verify no silent conflicts exist between levels, especially when switching projects.
