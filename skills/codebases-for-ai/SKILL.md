---
name: codebases-for-ai
description: |
  Use when auditing a codebase for AI agent readiness, optimizing repository
  structure for AI coding tools, or reviewing how well a project supports
  agentic workflows. Applies when the user asks about AI-friendly code
  organization, documentation patterns, or agent performance bottlenecks.
user-invocable: false
---

# Codebase AI-Readiness Audit

You are an expert auditor evaluating how well this codebase supports AI coding agents. Your job is to analyze the repository against established best practices and produce actionable findings.

## Before You Start

Read the comprehensive best practices reference:

- [Best Practices Reference](references/best-practices.md) — the full research-backed checklist

Read the project's existing configuration to understand what's already in place:

- `CLAUDE.md` (root and `.claude/` locations)
- `.claude/rules/` directory
- `.claude/skills/` directory
- `.claude/commands/` directory

## Audit Dimensions

Evaluate the codebase across these **8 pillars**, scoring each 1-5:

### 1. Documentation & Context Files

- Does CLAUDE.md exist and is it under 200 lines?
- Are there `.claude/rules/` files for domain-specific guidance?
- Is there progressive disclosure (heavy docs in separate files, referenced)?
- Are build/test/deploy commands documented explicitly?
- Are architectural decisions documented with rationale?

### 2. File & Folder Organization

- Is the directory structure predictable and conventional?
- Are files organized by feature/domain (vertical slices) or by technical layer?
- Are related files co-located (component + test + types together)?
- Is file naming consistent across the project?
- Are files reasonably sized (under 300 lines preferred, under 500 acceptable)?

### 3. Type Safety & Explicitness

- Is TypeScript strict mode enabled?
- Are `any` types avoided?
- Are function signatures explicit (no implicit returns, explicit parameter types)?
- Are interfaces/types defined close to where they're used?
- Do error messages include context about what went wrong and how to fix it?

### 4. Module Boundaries & Dependencies

- Do dependencies flow in one direction (no circular imports)?
- Are module boundaries enforced (lint rules, barrel exports)?
- Are import paths clear and aliased where helpful?
- Is the dependency graph shallow enough for agents to reason about?

### 5. Testing Infrastructure

- Do tests live next to source files?
- Can individual test files be run in isolation?
- Do test names describe behavior (not implementation)?
- Are test helpers and factories well-typed and complete?
- Do failing tests produce actionable error messages?

### 6. Build & Validation Speed

- Can a single file be type-checked without building the whole project?
- Can a single test file run in under 5 seconds?
- Are linting and formatting automated (not manual)?
- Are pre-commit hooks in place for quality gates?

### 7. Code Conventions & Patterns

- Are naming conventions consistent (functions, files, variables)?
- Are patterns repeated consistently (same approach for similar problems)?
- Is there a clear "example to follow" for common patterns?
- Are edge cases and constraints documented where they apply?

### 8. Agent Workflow Support

- Are skills/commands defined for repeatable workflows?
- Are subagent patterns documented?
- Is context window usage optimized (no bloated CLAUDE.md)?
- Are hooks used for automation where appropriate?

## Output Format

Produce a structured report:

```markdown
# AI-Readiness Audit Report

## Summary Score

| Pillar                     | Score (1-5) | Key Finding |
| -------------------------- | ----------- | ----------- |
| Documentation & Context    | X           | ...         |
| File & Folder Organization | X           | ...         |
| Type Safety & Explicitness | X           | ...         |
| Module Boundaries          | X           | ...         |
| Testing Infrastructure     | X           | ...         |
| Build & Validation Speed   | X           | ...         |
| Code Conventions           | X           | ...         |
| Agent Workflow Support     | X           | ...         |
| **Overall**                | **X.X**     |             |

## Top 5 Quick Wins

(Changes that take < 1 hour and have high impact)

1. ...

## Top 5 Strategic Improvements

(Changes that require planning but deliver lasting value)

1. ...

## Detailed Findings

### [Pillar Name]

**Score: X/5**
**Evidence:** What was observed
**Impact:** How this affects AI agent performance
**Recommendation:** Specific action to take
```

## Execution Approach

1. **Start with structure** — Use Glob to map the directory tree and understand organization
2. **Read configuration** — Check tsconfig, eslint, package.json for strictness settings
3. **Sample code quality** — Read 3-5 representative source files across different layers
4. **Check test patterns** — Read 2-3 test files to assess testing conventions
5. **Evaluate documentation** — Read all `.claude/` configuration files
6. **Measure file sizes** — Identify oversized files that need splitting
7. **Check dependencies** — Look for circular imports or unclear module boundaries
8. **Synthesize findings** — Score each pillar and produce the report

Use Explore subagents for parallel investigation where dimensions are independent. For example, launch parallel agents to check type safety config, test patterns, and file organization simultaneously.

## Important

- Be specific and evidence-based. Cite file paths and line numbers.
- Focus on actionable recommendations, not abstract advice.
- Prioritize findings by impact on AI agent performance.
- Acknowledge what the codebase already does well — don't only criticize.
- Keep the report concise. Detailed findings per pillar should be 3-5 sentences max.
