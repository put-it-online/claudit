# Context Window Reference

Expert guidance for managing Claude's 200K token context window in Claude Code environments.

## Composition Breakdown

| Component | Tokens | % of 200K |
|-----------|--------|-----------|
| System prompt | ~2,600 | 1.3% |
| Built-in tool definitions (Read, Write, Edit, Glob, Grep, WebSearch, etc.) | ~17,600 | 8.8% |
| MCP server schemas | 5,000–50,000+ | Variable (auto-defers at >10% overhead) |
| CLAUDE.md + .claude/rules/ | 1,000–5,000 | 0.5–2.5% |
| Compaction buffer | ~33,000 | 16.5% |
| **Available for work** | **~130,000** | **~65%** |

## Real Available Capacity

After fixed overhead (30–40K tokens) and compaction safety buffer (33K), approximately **130,000 tokens remain** for conversation, tool outputs, and file reads. This is your true capacity ceiling.

## Context Window Compaction

Compaction automatically triggers at ~95% of 200K window.

**What happens:**
- Earlier conversation summarized into a compact narrative
- Task state and decisions preserved in structured format
- CLAUDE.md and .claude/rules/ re-read from disk (always fresh)

**What survives:**
- Structured artifacts (plans, task lists, design docs)
- File changes and edit history
- Key decisions and rationale
- Function signatures and API contracts

**To re-inject critical context after compaction:**
Use SessionStart hook with `compact` matcher to restore specialized knowledge that didn't auto-survive.

## Performance Degradation Curve

| Usage | Behavior | Recommendation |
|-------|----------|-----------------|
| < 50% (100K) | Optimal recall and reasoning | Ideal working zone |
| 50–70% (100–140K) | Minimal degradation | Safe zone |
| 70–95% (140–190K) | Quality degrades noticeably | Start optimizing |
| ~95% (190K+) | Compaction triggers | Auto-summary mode |
| > 95% (200K) | Capacity exceeded | Compaction unavoidable |

**Critical insight:** Quality degrades around 147K tokens, not 200K. Treat **70% (140K) as practical ceiling** for sustained reasoning. Last 20% of context provides disproportionately poor value.

## Token Budget by Component

| What you control | Typical cost | Optimization |
|-----------------|-------------|--------------|
| CLAUDE.md | 500–2,000 | Keep under 150 lines; move detail to .claude/rules/ |
| .claude/rules/ files | 200–1,000 each | Path-specific loading; one file per domain |
| Skill metadata (all) | ~100 per skill | Concise descriptions; defer full content |
| Skill SKILL.md (when loaded) | 500–2,000 | Progressive disclosure: metadata first, reference files on-demand |
| MCP server schema | 5,000–10,000 per server | Auto-optimized by Tool Search at >10% overhead; selectively enable |
| File reads | 500–5,000 each | Read specific sections (use `offset` and `limit`); don't read entire files |
| Grep output | 200–2,000 | Use `-C` sparingly; prefer `output_mode: "files_with_matches"` |

## Optimization Techniques

**Progressive disclosure:** Load metadata at startup; defer full SKILL.md content to reference files. On-demand activation keeps base window small.

**Subagents:** Offload exploratory investigation and analysis to separate context windows. Agents get clean 200K capacity per invocation.

**Context reset:** Use `/clear` between unrelated tasks to reset window and recover capacity. Especially valuable when switching problem domains.

**Selective MCP servers:** Each costs 5–10K tokens at startup. Disable unused servers. Tool Search auto-defers servers at >10% window overhead.

**Structured artifacts:** Plans, task lists, and checklists survive compaction better than freeform notes. Use Plan Mode for complex work.

**Short CLAUDE.md:** Aim for <80 lines. Anything >200 lines gets truncated. Move architectural detail to .claude/rules/.

**Read selectively:** Use `offset` and `limit` parameters. Read 50–100 lines of a 2000-line file, not the whole file.

**Check token usage:** `/context` command shows real-time breakdown. Monitor when approaching 70% of window.

## Anti-Patterns to Avoid

- **Too many MCP servers:** 5 servers at 10K each = 50K tokens before conversation starts.
- **Verbose CLAUDE.md:** >200 lines gets truncated; >150 lines wastes space. Move to rules.
- **Redundant content:** Same guidance repeated in CLAUDE.md, rules, and skill docs.
- **Inline code examples:** Embed references to actual files instead of copying code blocks.
- **Ignoring 70% threshold:** Waiting until compaction at 95% means degraded performance for 25% of usage. Optimize earlier.
- **Not using `/clear`:** Context rot accumulates; conversation grows stale. Reset between task categories.
- **Loading all MCP servers:** Activate only the servers your task needs. Tool Search handles deferral intelligently.

## Decision Tree: When to Optimize

| Scenario | Action |
|----------|--------|
| Starting new task in same domain | Continue; context is relevant |
| Switching to unrelated problem | `/clear` to reset window |
| Approaching 70% (140K) | Begin selective reading, use subagents for investigation |
| At 85% (170K) | Reduce file reads, defer detail work |
| At 95% (190K+) | Compaction imminent; wrap up, prepare for reset |
| Need deep investigation | Spawn subagent with focused scope |

## Quick Checklist

- [ ] Keep CLAUDE.md under 150 lines
- [ ] Each .claude/rules/ file under 100 lines
- [ ] Skill SKILL.md defers detail to reference files
- [ ] Only 2–3 MCP servers active unless needed
- [ ] Monitoring `/context` output regularly
- [ ] Using `/clear` between unrelated tasks
- [ ] Reading files in sections, not whole
- [ ] Using structured artifacts for complex work
