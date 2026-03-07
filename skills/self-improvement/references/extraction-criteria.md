# Extraction Criteria

Shared reference for learning extraction and validation. Used by the self-improvement skill (extraction step) and the learning-curator agent (validation step).

## What To Look For

Reflect on the current session and identify insights in these categories:

- Unexpected system behavior
- Debugging discoveries
- Better implementation approaches
- Workflow improvements
- Tooling insights
- What would have saved time if known before starting

## Hard Filter

Only include items that meet **all three** conditions:

### Surprising

Not obvious from reading the code or documentation.

### Reusable

Applies beyond the specific task and could help future work.

### Actionable

Tells a future agent **what to do differently**, not just what exists.

## Formatting Rules

- **One heading per learning**
- **2–3 sentences maximum**
- **No case-specific details**
- **Expose a pattern** rather than describing a specific incident
- **Focus on guidance**, not implementation summary
- **Short and generic**

## Template

```
# Learning Title

2–3 sentences describing the insight. Explain what surprised you and what a future agent should do differently as a result. Make it generic enough, exposing a pattern rather than using case-specific details.
```

## Examples

Good:

> # Validate external API assumptions early
>
> External API behavior differed from documentation and only surfaced during integration. Always send a minimal real request early in a task to confirm behavior before implementing dependent logic.

> # Prefer incremental migrations over full rewrites
>
> Attempting a full replacement created cascading breakages that were difficult to isolate. Implement migrations in smaller stages so failures surface earlier and debugging remains localized.

Bad (do NOT include):

> # Added authentication middleware
>
> We added middleware to handle authentication across routes.

Why it's bad: not surprising, not reusable, just describes implementation.
