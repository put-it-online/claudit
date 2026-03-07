# CLAUDE.md Reference

Expert-level guidance on creating and maintaining CLAUDE.md files for Claude Code sessions.

## File Hierarchy & Loading

CLAUDE.md files load in precedence order. Most specific location wins.

| Location | Scope | When Loaded | Precedence |
|----------|-------|-------------|-----------|
| `~/.claude/CLAUDE.md` | Global, all projects | Session start | Lowest |
| `./CLAUDE.md` or `./.claude/CLAUDE.md` | Project root | Session start | Medium |
| `packages/web/CLAUDE.md` | Subdirectory/package | On-demand, when Claude reads files in that directory | Highest |
| `.claude/local.md` | Personal preferences | Session start | Not shared (gitignore) |

**Loading model**: Global file loads first. Project file merges/overrides global. Subdirectory file overrides project when Claude operates in that context.

**Precedence rule**: When multiple files exist, Claude uses the most specific location. A subdirectory CLAUDE.md takes priority over project-level, which takes priority over global.

---

## File Sizing & Truncation

Content beyond 200 lines is silently truncated. Plan accordingly.

| Lines | Compliance Rate | Recommendation |
|-------|-----------------|-----------------|
| 80-150 | 98% | Target zone. Most instructions applied. |
| 150-200 | 95% | Acceptable. Minor truncation risk. |
| 200-400 | 71% | Degraded. Half of content ignored. |
| 500+ | ~50% | Critical. Most rules lost to truncation. |

**Strategy**: Target 80-150 lines. Use progressive disclosure — link to `.claude/rules/` subdirectory for domain-specific depth (architecture.md, testing.md, schemas.md).

**MEMORY.md**: Also 200-line limit. For overflow topics, create supplementary files alongside MEMORY.md at `~/.claude/projects/<project>/memory/debugging.md`, `patterns.md`, etc., and reference from MEMORY.md.

---

## Structure: WHY, WHAT, HOW

Frame CLAUDE.md as three sections. Front-load critical information.

```
WHY (1-2 lines)
  → What type of project is this? What makes it different?

WHAT (2-3 sections)
  → Tech stack, key directories, package structure
  → Build/test/dev commands Claude can't guess
  → Key packages and imports

HOW (Conventions + Common Gotchas)
  → Dev workflow, code style (beyond linters)
  → Architecture decisions unique to this project
  → Gotchas, workarounds, verification steps
  → Reference .claude/rules/ for deeper topics
```

**Example opening**:
```markdown
# Architectural Model

This is a **pnpm monorepo** following a simplified **Clean Architecture**.
Dependencies flow outer → inner only.

### Layers
[table of layers]

### Key Packages
[table of imports and their sources]
```

---

## CLAUDE.md vs MEMORY.md

Two complementary knowledge bases.

| Aspect | CLAUDE.md | MEMORY.md |
|--------|-----------|-----------|
| **Owner** | You (project maintainer) | Claude (auto-generated) |
| **Location** | Project root or `./.claude/` | `~/.claude/projects/<project>/memory/` |
| **Content** | Project instructions, design decisions, setup rules | Session learnings, verification results, patterns discovered |
| **When updated** | PRs, architecture changes | After each session |
| **Lifespan** | Project lifetime | Cumulative across sessions |
| **Size limit** | 200 lines | 200 lines (use topic files for overflow) |

**Workflow**: CLAUDE.md is your source of truth. MEMORY.md accumulates Claude's learnings (test patterns, deployment gotchas, recurring fixes). When a MEMORY.md pattern becomes stable, migrate it to CLAUDE.md via PR and remove from MEMORY.md.

**Topic files**: For MEMORY.md overflow, create `~/.claude/projects/<project>/memory/patterns.md`, `debugging.md`, etc., and reference them from MEMORY.md index. Each topic file has its own 200-line budget.

---

## What to Include

**Included in CLAUDE.md**:
- 1-2 line project description (what makes it different)
- Directory structure map (key packages, their purpose)
- Build/test/dev commands Claude can't infer
- Architecture decisions (layers, dependency rules, no-global-state rule)
- Code conventions that differ from language defaults
- Common gotchas, workarounds, verification steps
- Non-obvious tool configurations (monorepo setup, build order, etc.)

**Excluded from CLAUDE.md**:
- Formatting rules (ESLint/Prettier handle these)
- Language fundamentals Claude infers from code
- Standard patterns (use `grep` or file references instead)
- Detailed API documentation (link to files or comments)
- File-by-file descriptions (too verbose)
- Large code examples (reference actual source files)
- README duplicates (CLAUDE.md is for workflow, not onboarding)

---

## Common Mistakes

**Over 200 lines**: Silent truncation. Content beyond line 200 is lost. Target 80-150 lines instead.

**Inlining code examples**: High token cost, low value. Always reference actual source files: "See `packages/web/src/useTheme.tsx` for type narrowing patterns." File reads are cached and cheaper.

**Duplicating README**: README onboards new contributors. CLAUDE.md guides Claude's workflow. Separate concerns.

**Formatting rules in CLAUDE.md**: Prettier and ESLint enforce style. Don't waste lines on rules that tooling already enforces. Use CLAUDE.md for build order, monorepo gotchas, non-obvious conventions.

**Task-specific rules at root level**: If a rule applies to only one package or subsystem, put it in `packages/web/CLAUDE.md`, not the root. Root CLAUDE.md is for cross-cutting concerns.

**Not using `/init`**: Start CLAUDE.md work with `/init` command to load context. Ensures your instructions are properly framed.

---

## Writing Tips

- **Front-load**: Critical information in first 20 lines.
- **Tables**: Use for architecture layers, commands, file locations. More scannable than prose.
- **Links**: Reference `.claude/rules/architecture.md` for deep dives. Saves lines.
- **Progressive disclosure**: High-level summary in CLAUDE.md, detailed rules in `.claude/rules/` subdirectory.
- **Gotchas first**: Common mistakes and verification steps are more valuable than theoretical explanations.
- **Commands as-is**: Include exact commands (e.g., `pnpm --filter @pkg build`). Don't paraphrase.

---

## Precedence Quick Reference

When editing CLAUDE.md:
1. Is this a formatting rule? → Use ESLint/Prettier, not CLAUDE.md.
2. Is this project-wide? → Keep in root CLAUDE.md.
3. Is this package-specific? → Move to `packages/web/CLAUDE.md`.
4. Is this a deep-dive topic? → Put in `.claude/rules/topic.md`, link from CLAUDE.md.
5. Is this a session learning? → Goes to MEMORY.md or topic files in `~/.claude/projects/<project>/memory/`.
6. Does this duplicate README? → Remove from CLAUDE.md, refer to README instead.
