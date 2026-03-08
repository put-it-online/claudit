# Clauditor

Master your Claude Code setup with config auditing, automated learning capture, and deep concept knowledge.

## Installation

### Claude Code (via Plugin Marketplace)

Register the marketplace:

```bash
/plugin marketplace add put-it-online/claudit
```

Install the plugin:

```bash
/plugin install clauditor@claudit
```

### Verify Installation

Start a new Claude Code session and run:

```
/clauditor:claudit
```

## What's Inside

### Commands

- **`/clauditor:claudit`** - Audit and optimize your Claude Code configuration (CLAUDE.md, rules, hooks, skills, agents, memory, settings)
- **`/clauditor:audit-codebase-for-ai`** - Audit the codebase for AI agent readiness — evaluates documentation, structure, types, testing, and workflow support across 8 pillars
- **`/clauditor:learn`** - Capture reusable learnings from your current session

### Skills

- **self-improvement** - Extract surprising, reusable, actionable insights from sessions and store them in the right location
- **claude-concept-expert** - Deep reference knowledge for CLAUDE.md, skills, hooks, rules, subagents, context window, and workflows
- **codebases-for-ai** - Audit a codebase for AI agent readiness across 8 pillars (documentation, structure, types, modules, testing, build speed, conventions, workflow support)

### Agents

- **config-auditor** - Full configuration audit with expert knowledge, scoped findings by severity, and guided one-at-a-time fixes
- **learning-curator** - Evaluates candidate learnings, checks for duplicates, and recommends storage destinations
- **knowledge-synthesizer** - Extracts actionable patterns from agent interactions and builds collective intelligence

## How It Works

### Auditing (`/clauditor:claudit`)

1. Discovers all Claude config files across project, memory, and global scopes
2. Audits each file against best practices using the claude-concept-expert knowledge base
3. Presents a full report with scoped and severity-tagged findings
4. Walks through each actionable finding one at a time, asking for approval before applying changes

### AI-Readiness Audit (`/clauditor:audit-codebase-for-ai`)

1. Maps the directory tree and reads configuration files (tsconfig, eslint, package.json)
2. Samples representative source and test files across the codebase
3. Scores the project across 8 pillars: documentation, structure, types, modules, testing, build speed, conventions, and agent workflow support
4. Produces a structured report with per-pillar scores, top 5 quick wins, and top 5 strategic improvements

### Learning (`/clauditor:learn`)

1. Extracts candidate learnings from the current session (surprising, reusable, actionable)
2. Delegates to the learning-curator agent for duplicate checking and destination recommendations
3. Presents curated list for bulk approval
4. Writes approved learnings to the recommended destinations

## License

MIT License - see LICENSE file for details.
