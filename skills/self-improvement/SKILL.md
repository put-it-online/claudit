---
name: self-improvement
description: |
  Document reusable learnings. Use this skill immediately before completing
  a planning or execution session. Captures insights from the current context
  window and stores them in the right location.
user-invocable: false
---

# Self Improvement

## Purpose

Capture reusable lessons from the current context window so future agents benefit from discoveries made during planning or implementation.

## When To Use

Use this skill **immediately before completing any session** — planning or execution.

Typical triggers:

* A planning or brainstorming session is wrapping up
* A plan has been executed or implemented
* The task is about to be marked as finished
* The session produced insights worth preserving

---

# Instructions

## 1. Extract Candidate Learnings

Reflect on the current session and extract candidate learnings.

Read the full extraction criteria from [references/extraction-criteria.md](references/extraction-criteria.md) before proceeding. Apply the hard filter: every candidate must be **surprising**, **reusable**, and **actionable**.

Follow the formatting rules exactly: one heading per learning, 2–3 sentences, no case-specific details, expose a pattern, focus on guidance.

---

## 2. Delegate to Learning Curator

Send the candidate list to the `@learning-curator` agent.

Format your delegation message as:

```
Here are the candidate learnings from this session. Evaluate each one
and recommend storage destinations.

## Candidates

### 1. [Learning Title]

[2-3 sentence description]

### 2. [Learning Title]

[2-3 sentence description]

...
```

The curator will:
- Read existing CLAUDE.md, `.claude/rules/`, and auto-memory files
- Check for duplicates
- Re-validate against the extraction criteria
- Recommend a destination for each learning
- Suggest wording refinements

Wait for the curator's response before proceeding.

---

## 3. Present Curated List to User

After receiving the curator's recommendations, present the full list to the user for bulk approval.

Format:

```
Learnings captured from this session:

[1] Learning title
    → destination/path.md (rationale)

[2] Learning title
    → destination/path.md (rationale)

[3] DUPLICATE: Learning title
    → Already in destination/path.md — skip

Reply with numbers to remove (e.g., "1,3"), or "approve" to write all non-duplicate items.
"none" to skip all.
```

Rules:
- Duplicates are flagged and auto-excluded (user can override)
- User can remove individual items by number
- "approve" writes all remaining non-duplicate items
- "none" aborts — nothing is written

---

## 4. Write Approved Learnings

For each approved learning, write it to the curator's recommended destination.

Writing rules:
- If the destination file exists, append under the appropriate section
- If the destination file does not exist, create it
- For `.claude/CLAUDE.md`, respect the 200-line hard limit — if adding would exceed it, use a `.claude/rules/` file instead
- For auto-memory files, follow the existing semantic organization
- Do **not** duplicate learnings that already exist in the destination

---

## 5. Commit

Commit all changes with a descriptive message.

```bash
git add <all modified files>
git commit -m "docs: capture session learnings"
```
