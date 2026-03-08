# AI-Friendly Codebase Best Practices

Comprehensive reference compiled from 60+ sources (2024-2026). Use this as the evaluation baseline for audits.

## The Five Pillars

1. **Explicit Knowledge** — Remove implicit assumptions through clear documentation, structured information, and explicit constraints
2. **Efficient Navigation** — Organize files predictably, use dependency graphs, co-locate related code
3. **Strong Typing** — Use strict TypeScript with clear contracts and explicit interfaces
4. **Modular Architecture** — Small, focused modules organized by feature (vertical slices)
5. **Strategic Tooling** — Monorepo structure with dependency visibility, lightweight validation tools, clear automation paths

---

## 1. Documentation & Context Files

### CLAUDE.md / AGENTS.md

- Acts as persistent memory, read at the start of every session
- Should be **under 200 lines** (ideally <80 lines for core content)
- Manually crafted, not auto-generated — "the highest leverage point of the harness"
- Content: tech stack, project purpose, build/test commands, design constraints
- Avoid: extensive style guides (use linters), duplicate README info, task-specific instructions
- LLMs can follow ~150-200 instructions consistently; more causes degradation

### Progressive Disclosure

- Store detailed guidance in separate markdown files
- Reference from CLAUDE.md without embedding full content
- Let agents decide which docs are relevant and read on-demand
- Heavy reference files belong in supplementary locations, not CLAUDE.md

### What to Document Explicitly

- Build and test commands with exact syntax
- Architectural decisions and their rationale
- Common pitfalls and constraints
- File structure overview with key locations
- Edge cases that require special handling
- "Dos and Don'ts" — be as nitpicky as needed

### Real Examples Over Abstractions

- Point to specific well-written components to emulate
- Reference legacy files to avoid
- Show one real code snippet that demonstrates style (better than three paragraphs describing it)

---

## 2. File & Folder Organization

### Directory Structure

- Organize into conventional directories: `components/`, `lib/`, `hooks/`, `types/`, `services/`
- Use consistent naming conventions (kebab-case for files, camelCase for functions)
- Place configuration files at project root for immediate agent access
- Co-locate relevant docs alongside code

### Vertical Slice Architecture

- Organize by business capabilities rather than technical layers
- Each feature in its own directory: handler, DTOs, validators, co-located tests
- Maximizes coupling within a feature, minimizes coupling between features
- Prevents 2000-line services; features remain additive and small

### File Size Guidelines

- **Under 300 lines**: Ideal for AI comprehension
- **300-500 lines**: Acceptable but consider splitting
- **Over 500 lines**: Integration width bottleneck — split into focused modules
- Functions under 50-100 lines for best AI performance
- Large files create DFI (Distribution of Function Integration) bottlenecks

### Context Window Impact

- Processing 128K tokens takes significantly longer than 4K
- Accuracy drops sharply with context size (Claude 3.5: 29% at 32K → 3% at 256K on LongSWE-Bench)
- Maximum Effective Context Window is drastically different from advertised maximum
- Goal: structure code so context needed per task is minimal

---

## 3. Type Safety & Explicitness

### TypeScript Configuration

- Enable strict mode: `noImplicitAny`, `strictNullChecks`, explicit return types
- AI agents are 9x more prone to use `any` than humans
- Teams that defined strict types saw AI generate more accurate code in fewer attempts
- Type safety improves AI success rates from ~60% to nearly 100%
- Each strict constraint eliminates a decision point where AI could generate ambiguous code

### Self-Documenting Code

- Descriptive names that encode intent without requiring comments
- Consistent patterns (boolean functions: `is`, `has`, `can` prefixes)
- Replace conditionals with named functions that encapsulate logic
- For AI agents: favor explicit clarity, consistency, verbose naming, regular structure

### Error Messages

- AI agents perform better with verbose text descriptions containing complete details
- Include: what went wrong, attempted values, expected constraints, fix suggestions
- Stack traces should be complete, not truncated
- Structured diagnostic tools enable targeted code changes

### API Contracts

- Generate TypeScript types from OpenAPI specs
- Typed API client libraries reduce malformed requests
- Compile-time type checking catches 73% of agent configuration errors before deployment

---

## 4. Module Boundaries & Dependencies

### Dependency Management

- Dependencies flow outer → inner only (Clean Architecture)
- Explicit import/export statements indicating architectural intent
- No circular dependencies — creates confusion in agent reasoning
- Use path aliases for clarity (`@app/`, `@services/`)
- Barrel exports (index.ts) to control what's exported

### Dependency Graph

- Expose workspace structure directly to agents (don't make them grep)
- Agents query dependency graphs to find impacted projects, analyze breaks, produce fix plans
- Structured metadata per project: available tasks, cache inputs/outputs, configuration

### Monorepo Advantages for AI

- AI agents see the entire system at once — understanding cross-cutting changes
- Properties that slow humans down help AI agents operate more effectively
- Monorepo PR cycle times run 9x slower at median but offset by AI advantage
- In polyrepos, agents see one project at a time, blind to ripple effects

### Anti-Patterns

- Circular dependencies creating DAG violations
- God files with high integration width (DFI bottleneck)
- Implicit soft dependencies not visible in import graph
- Single-file manifests that don't scale beyond 100K LOC

---

## 5. Testing Infrastructure

### Test Structure

- Tests live next to source files (`*.test.ts`, `*.test.tsx`)
- Agents locate and understand test expectations immediately
- Test names describe behavior, not implementation
- Edge cases encoded explicitly in test suites

### Coverage as Behavioral Gate

- Code coverage effective as regression signal (not vanity metric)
- Strict coverage gates on PRs make attempts to weaken checks visible
- High-threshold gates (85-95%+) surface regressions early
- 83.8% of agent-assisted PRs eventually accepted; 54.9% merged without modification

### Test Quality

- Well-typed test helpers and factories (verify against entity types)
- Failing tests produce actionable error messages
- Tests validate function contracts and invariants clearly
- Probabilistic assertions for AI-generated code where deterministic tests fail

### Hook Test Environment

- Hook tests (`.test.ts`) require explicit environment configuration
- The `.tsx` glob alone may be insufficient for hook test environments
- Document environment configuration requirements explicitly

---

## 6. Build & Validation Speed

### Per-File Operations

- Provide lightweight type-checking for single files
- Single-file linting operations
- Targeted test runs for specific modules
- Avoid forcing 30-60 second full builds for simple checks

### Linting as AI Control

- Use linters to direct and constrain code generation
- Seven categories: grep-ability, glob-ability, architecture, security, testability, observability, documentation
- Apply stricter rules selectively to AI-generated code
- Run type-checkers after AI generation, formatters before

### Build Pipeline

- Pre-commit hooks for quality gates
- CI/CD with semantic evaluation (not exact string matching)
- Progressive deployment with automated rollback
- Fast feedback loops (build/test/deploy under 30 seconds)

### Quality Impact

- AI-co-authored PRs contain 1.7x more issues than human-only code
- 75% more logic errors, 3x more readability problems, 2x more error handling gaps
- With stricter linting, these gaps narrow significantly

---

## 7. Code Conventions & Patterns

### Consistency

- Consistent naming throughout (all boolean functions start with `is`, `has`, `can`)
- Same approach for similar problems (don't mix patterns)
- Avoid abbreviations unless universally understood
- Single Responsibility Principle: one module, one clear purpose

### Pattern Documentation

- Point to exemplar files for each pattern
- Document "why" behind architectural decisions
- Include anti-patterns to avoid (with references to legacy code)

### Cognitive Complexity

- Functions with cognitive complexity above 15 require significant mental effort
- AI-generated code may have low cyclomatic but high cognitive complexity
- Agents create code with single-line operations nesting multiple concerns
- Target: flattened conditional structures, discrete composable units

### Code Duplication

- Code duplication percentage serves as context blindness indicator
- Without right context: duplication 40% higher, cognitive complexity 43% higher
- Point agents toward existing abstractions to prevent regeneration
- DRY enforcement through thoughtful context design

---

## 8. Agent Workflow Support

### Skills & Commands

- Skills: model-invoked (auto-triggered based on context)
- Commands: user-invoked (explicit slash command)
- One skill per workflow (e.g., `create-feature`, `debug-test-failure`)
- Store in `.claude/skills/` (project) or `~/.claude/skills/` (personal)

### Subagent Patterns

- Delegate specialized tasks to focused agents
- Enable parallel execution of independent work
- Narrow tool scope for each subagent
- Use worktree isolation for parallel development

### Context Window Optimization

- CLAUDE.md: 15-50K tokens (keep under 80 lines)
- Each rule file: 5-15K tokens (load on-demand)
- Each MCP server: 5-10K tokens at session start
- If total > 100K before first prompt, trim aggressively

### Hooks

- PostToolUse for auto-formatting after edits
- SessionStart for context recovery after compaction
- Use matchers to narrow scope (don't fire on every event)
- Let linters handle formatting; hooks should orchestrate

---

## Metrics That Correlate with AI Success

| Metric               | Target            | Why                              |
| -------------------- | ----------------- | -------------------------------- |
| File size            | < 300 lines       | Reduces context needed per task  |
| Function length      | < 50-100 lines    | Better AI comprehension          |
| Cognitive complexity | < 15 per function | Lower reasoning burden           |
| Code duplication     | < 5%              | Signals good context             |
| Test coverage        | > 85%             | Behavioral regression gate       |
| CLAUDE.md size       | < 200 lines       | Prevents instruction degradation |
| Type coverage        | > 95%             | Eliminates ambiguity             |
| Build feedback       | < 30 seconds      | Fast iteration loop              |

## Signs of Good AI-Friendly Structure

- Agents find relevant code in < 3 file reads
- New features require < 20% code changes vs. existing patterns
- Test failures provide clear, actionable feedback
- Agents don't ask "where should I put this?" — structure is obvious
- Token usage stays consistent across similar-sized tasks
- Agents make minimal exploratory searches

## Common Anti-Patterns

| Anti-Pattern              | Impact                         | Fix                                      |
| ------------------------- | ------------------------------ | ---------------------------------------- |
| CLAUDE.md > 200 lines     | Instruction-following degrades | Move domain sections to `.claude/rules/` |
| Implicit conventions      | Agents can't infer them        | Make all knowledge explicit              |
| God files (> 500 lines)   | Integration width bottleneck   | Split into focused modules               |
| Missing type definitions  | 9x more `any` usage            | Enable strict TypeScript                 |
| No per-file test commands | Forces full builds             | Add targeted test scripts                |
| Circular dependencies     | Agent reasoning confusion      | Enforce one-way dependency flow          |
| Scattered documentation   | No single source of truth      | Consolidate in `.claude/` hierarchy      |
| Duplicated rules          | Wasted tokens every session    | Audit for overlap, delete one copy       |

---

## Sources

This reference was compiled from 60+ sources including:

- Anthropic's Claude Code Best Practices
- Factory.ai Agent Readiness Framework (8 pillars x 5 maturity levels)
- AGENTS.md Open Standard (adopted by 60,000+ projects)
- ArXiv papers: Codified Context, Theory of Code Space, Repository-Level Reasoning
- JetBrains, Martin Fowler, CodeScene, Augment Code engineering blogs
- Real-world case studies from teams restructuring codebases for AI agents
