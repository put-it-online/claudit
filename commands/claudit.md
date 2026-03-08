---
description: "Audit and optimize the Claude Code configuration for this project."
disable-model-invocation: true
---

Delegate this task to @config-auditor. It has the `claude-concept-expert` skill loaded for in-depth knowledge of Claude concepts.

The agent will:
1. Discover all Claude config files (project, memory, and global scope)
2. Audit each file against best practices
3. Present a full report with scoped and severity-tagged findings
4. Walk through each actionable finding one at a time, asking for approval before applying changes
