# Hooks Reference

User-defined shell commands, HTTP endpoints, or LLM prompts that execute at specific lifecycle points in Claude Code. Hooks provide deterministic control—guaranteed to fire at exact moments, unlike LLM instructions which depend on model behavior. Use them for security gates, audit logging, context recovery, and automation that must always execute.

## Event Reference

| Event | Fires when | Matcher filters | Key use |
|-------|-----------|----------------|---------|
| `SessionStart` | Session begins or resumes | startup, resume, clear, compact | Context injection, setup, recovery after compaction |
| `UserPromptSubmit` | User submits a prompt | — | Audit logging, context enrichment |
| `PreToolUse` | Before tool executes | tool name | Block dangerous operations, enforce policy |
| `PermissionRequest` | Permission dialog shown | tool name | Auto-approve/deny, custom gates |
| `PostToolUse` | Tool succeeds | tool name | Auto-format, cleanup, logging |
| `PostToolUseFailure` | Tool fails | tool name | Error recovery, remediation |
| `Notification` | Claude needs input | notification type | Desktop alerts, escalation |
| `SubagentStart` | Subagent spawns | agent type | Environment setup, context passing |
| `SubagentStop` | Subagent finishes | agent type | Cleanup, result collection |
| `Stop` | Claude finishes task | — | Final validation before completion |
| `TaskCompleted` | Task marked done | — | Verification, reporting |
| `InstructionsLoaded` | CLAUDE.md/rules parsed | — | Debug, audit, reload detection |
| `ConfigChange` | Config modified | config source | Audit, block unauthorized changes |
| `PreCompact` | Before context compaction | manual, auto | Save state, preserve context |
| `WorktreeCreate` | Worktree created | — | Custom setup, bootstrapping |
| `WorktreeRemove` | Worktree removed | — | Cleanup, validation |
| `SessionEnd` | Session ends | clear, logout, etc. | Final cleanup, archiving |

## Hook Types

| Type | How it works | Best for | Input source | Response handling |
|------|-------------|----------|--------------|-------------------|
| `command` | Shell command, stdin/stdout | Validation, formatting, logging | JSON on stdin | Exit code: 0=allow, 2=block, other=log |
| `prompt` | Single-turn LLM call (Haiku default) | Yes/no decisions, judgment | Formatted prompt | LLM response in Claude context |
| `agent` | Subagent with full tool access (up to 50 turns) | File inspection, complex verification | Full environment | Structured JSON response |
| `http` | POST to external URL endpoint | Shared audit, external systems | JSON body | HTTP response code + body |

## Configuration Locations

| Location | Scope | Checked into git |
|----------|-------|-----------------|
| `~/.claude/settings.json` | All projects | No (user-specific) |
| `.claude/settings.json` | Current project | Yes |
| `.claude/settings.local.json` | Current project | No (git-ignored) |
| Managed policy | Organization-wide | Yes (admin-managed) |
| Plugin `hooks/hooks.json` | When plugin enabled | Yes |
| Skill/agent frontmatter | While skill active | Yes |

## Hook Input/Output Protocol

**Input (JSON on stdin):**
```json
{
  "session_id": "sess_abc123",
  "cwd": "/path/to/project",
  "hook_event_name": "PreToolUse",
  "tool_name": "Edit",
  "tool_input": { "file_path": "/path/file.ts", ... },
  "event_context": { "matcher_value": "Edit", ... },
  "stop_hook_active": false
}
```

**Exit codes:**
- **0**: Proceed (stdout added to Claude context for certain events)
- **2**: Block/Deny (stderr becomes Claude feedback)
- **Other**: Proceed but log (stderr logged, hidden from Claude)

**Structured response (JSON on stdout):**
```json
{
  "hookSpecificOutput": {
    "permissionDecision": "deny",
    "permissionDecisionReason": "File matches blocked pattern: .env"
  }
}
```

## Common Patterns

**Auto-format after edits**
```json
{
  "type": "command",
  "on": "PostToolUse",
  "matcher": { "tool_name": ["Edit", "Write"] },
  "command": "prettier --write $TOOL_INPUT_file_path"
}
```

**Block dangerous files**
```json
{
  "type": "command",
  "on": "PreToolUse",
  "matcher": { "tool_name": "Edit" },
  "command": "! echo $TOOL_INPUT_file_path | grep -E '(\\.env|\\.git/|node_modules)'"
}
```

**Recover context after compaction**
```json
{
  "type": "command",
  "on": "SessionStart",
  "matcher": { "startup_type": "compact" },
  "command": "cat ~/.claude/critical-context.txt"
}
```

**Audit all tool use**
```json
{
  "type": "command",
  "on": "PostToolUse",
  "command": "echo \"[$(date)] $HOOK_EVENT_NAME: $TOOL_NAME\" >> /tmp/audit.log"
}
```

**Verify task completion**
```json
{
  "type": "prompt",
  "on": "Stop",
  "prompt": "Verify all requested tasks in the conversation are complete. Respond only: YES or NO."
}
```

## Common Mistakes

**Stop hook infinite loop**
If a Stop hook runs PreToolUse logic that itself triggers Stop, infinite recursion occurs. Guard with:
```bash
if [[ $(jq -r '.stop_hook_active' <<< "$input") == "true" ]]; then
  exit 0
fi
```

**Shell profile echo breaks JSON**
Login shell initialization (`.bashrc`, `.zshrc`) may echo banner text. Wrap output in:
```bash
if [[ $- == *i* ]]; then
  echo "Interactive shell message"
fi
```

**Missing execute permission**
Hooks must be executable:
```bash
chmod +x ~/.claude/hooks/my-hook.sh
```

**PermissionRequest in non-interactive mode**
`PermissionRequest` dialogs don't appear in batch mode (`-p` flag). Use `PreToolUse` with `permissionDecision: "deny"` instead.

**Untested shell escaping**
Before deploying:
```bash
echo '{"tool_name":"Edit","tool_input":{"file_path":"/path/with spaces.ts"}}' | ./hook.sh
```

**Relying on environment variables set by Claude**
Only use variables passed in JSON input. Shell environment is not guaranteed across hook invocations.

## Testing Hooks Locally

**Test a command hook:**
```bash
cat > /tmp/test-hook.json << 'EOF'
{
  "session_id": "test",
  "cwd": "/path/to/project",
  "hook_event_name": "PreToolUse",
  "tool_name": "Edit",
  "tool_input": { "file_path": "test.ts" },
  "stop_hook_active": false
}
EOF
cat /tmp/test-hook.json | ~/.claude/hooks/my-hook.sh
```

**Verify JSON is valid:**
```bash
echo '{"hookSpecificOutput":{"permissionDecision":"deny"}}' | jq .
```

**Check hook registration:**
```bash
claude settings show hooks
```

## Advanced: Hooks + Matchers

Matchers filter which hook invocations fire. Common matchers:

- `tool_name`: Array of tool names (e.g., `["Edit", "Write"]`)
- `startup_type`: One of `startup`, `resume`, `clear`, `compact`
- `notification_type`: One of `input_required`, `confirmation`, etc.
- `agent_type`: Agent class name
- `config_source`: One of `.claude/settings.json`, `~/.claude/settings.json`, etc.

Use matchers to avoid unnecessary hook invocations—each hook call has latency overhead.

## Advanced: Agent Hooks

Agent hooks run a subagent (up to 50 turns, full tool access) and capture structured JSON from its final response. Use for complex decisions:

```json
{
  "type": "agent",
  "on": "Stop",
  "agent": "verification-expert",
  "prompt": "Review the conversation. Has every task been completed? Return JSON: {\"allTasksComplete\": true|false, \"missingTasks\": [...]}"
}
```

The agent's final `{...}` JSON block is extracted and available to Claude as context.
