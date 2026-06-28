---
name: update-claude-md
description: >
  Updates CLAUDE.md to reflect the current state of the project based on
  recent actions and session history, without touching protected core rules,
  code style conventions, or project goals. Use this skill whenever the user
  runs /update-claude-md, asks to "sync CLAUDE.md", "update project context",
  "reflect recent changes in CLAUDE.md", or when significant planning vs
  execution divergence is detected. Also invoke after major refactors,
  architecture pivots, or completed milestones.
---

# Update CLAUDE.md

Keeps CLAUDE.md accurate by syncing context sections with what has actually
been built, while preserving protected zones that define unchanging project
rules and goals.

---

## The protected zone contract

CLAUDE.md sections wrapped in `<!-- PROTECTED -->` markers must **never** be
modified by this skill, regardless of what has changed in the codebase:

```markdown
<!-- PROTECTED -->
## Code Style

- Use black formatting, 88 char line length
- All public functions must have docstrings
<!-- /PROTECTED -->
```

Protected sections typically contain:
- Code formatting rules
- Testing requirements
- Security constraints
- Core project goals / north star
- Architectural mandates

If these sections appear outdated, **flag them** (see Flagging below) but do
not change them. The human must explicitly unlock them.

---

## Step-by-step

### 1. Read and map CLAUDE.md

Read the full CLAUDE.md. Build a mental map of:
- Which sections are `<!-- PROTECTED -->` → off-limits entirely
- Which sections describe **project state** (architecture, current sprint,
  module status, known issues) → eligible for update
- Which sections are the `## Session Log` → managed by `/save-session`, do not
  touch the table rows (you may add a note below the table if useful)

### 2. Read recent session files

```bash
ls -lt claude-sessions/ | head -5
```

Read the 3 most recent session summary files. Extract:
- Solutions Implemented (things now done that CLAUDE.md may not reflect)
- Problems Identified (new known issues)
- Deferred Work (open TODOs)
- Any explicit mentions of architecture or convention changes

### 3. Diff intent vs reality

For each updatable section in CLAUDE.md, compare what it says against what the
session files reveal. Flag any of these conditions:

| Condition | Action |
|---|---|
| Section is **accurate** | No change needed |
| Section has **minor drift** (small additions, completed items) | Update inline |
| Section has **significant drift** (feature removed, module renamed) | Update + note the change in your report |
| Section contradicts a **protected zone** | FLAG — do not resolve automatically |
| Session files show **planning vs execution divergence** | FLAG with a diff summary |

### 4. Detect planning vs execution divergence

This is the most important check. A divergence occurs when:

- A planned approach was abandoned in favour of a different implementation
- A module or component described in CLAUDE.md no longer exists or was renamed
- A dependency was swapped out (e.g. replaced `requests` with `httpx`)
- An architectural decision was reversed (e.g. "we went sync instead of async")
- Scope was significantly reduced or expanded without updating CLAUDE.md

**Threshold for flagging:** If the divergence would mislead a new Claude
instance reading CLAUDE.md cold, it must be flagged.

### 5. Apply updates

For each eligible section with changes:

1. Make the targeted edit — change only what has drifted, preserve everything else
2. Do not rewrite sections in your own words if the existing wording is accurate
3. Keep additions concise — CLAUDE.md is a navigation document, not a log

**Typical updates you will make:**

- Mark completed items in a task/milestone list with `✓`
- Add newly discovered constraints to "Known Issues" or "Gotchas"
- Update module/file names that were renamed
- Remove references to deleted components
- Add new modules/services that were introduced this session

### 6. Never do these things

- ❌ Rewrite the whole file
- ❌ Change the tone or structure of protected sections
- ❌ Add editorial commentary ("This section could be improved…")
- ❌ Remove the `## Session Log` table or its rows
- ❌ Alter code style rules even if you think they're wrong
- ❌ Resolve a flagged divergence without user confirmation

### 7. Report changes

After applying updates, print a structured report:

```
## CLAUDE.md Update Report

### Changes applied
- [Architecture] Updated `DataPipeline` module name → `IngestPipeline` (renamed in session 2025-10-15)
- [Known Issues] Added: "Batch flush drops last record when n < chunk_size"
- [Milestones] Marked Phase 1 ingest as ✓ complete

### No change needed
- Code Style (PROTECTED — skipped)
- Testing Requirements (PROTECTED — skipped)
- Project Goals (accurate)

### ⚑ FLAGS requiring your attention
- [Divergence] CLAUDE.md describes async pipeline; sessions show sync implementation was used instead
  → See: claude-sessions/2025-10-14-wikimedia-ingest-pipeline-debugging.md
  → Recommended: update Architecture section to reflect sync design
  → To apply: confirm with "yes, update architecture to sync"
```

---

## Flagging

When a condition requires human judgment, emit a Flag block and stop:

```
⚑ FLAG — [brief title]

Situation: [what was found, with specifics]
Impact: [what a future Claude session would get wrong if this isn't resolved]
Options:
  A. [option A — e.g. update CLAUDE.md to reflect actual implementation]
  B. [option B — e.g. revert code to match original plan]
  C. Leave as-is and I'll handle it manually

Waiting for your instruction before proceeding.
```

**Always flag, never guess, for:**
- Planning vs execution divergence in architecture or core decisions
- Any change that would touch a `<!-- PROTECTED -->` section
- Contradictory information across session files (which version is canonical?)
- A section the skill cannot confidently update without domain context

---

## Adding PROTECTED markers to CLAUDE.md

If CLAUDE.md has no protected markers yet, suggest adding them:

> "CLAUDE.md has no `<!-- PROTECTED -->` zones. Want me to add markers around
> the Code Style, Testing, and Project Goals sections so future updates can't
> accidentally change them? Reply 'yes, add markers' to proceed."

Do not add markers without explicit confirmation.

---

## Edge cases

**claude-sessions/ doesn't exist or is empty**
Skip Step 2. Base updates only on the current conversation and CLAUDE.md
content. Note in the report: "No session files found — update based on
current conversation only."

**CLAUDE.md doesn't exist**
Do not create it. Tell the user: "CLAUDE.md doesn't exist yet. Run
`/save-session` first to bootstrap it, or create a CLAUDE.md manually and
then rerun `/update-claude-md`."

**No eligible sections to update**
Report: "CLAUDE.md appears up to date — no changes applied." Still run the
divergence check and flag anything found.

**User asks to update a protected section**
Acknowledge the request, explain the protection, and offer to remove the
`<!-- PROTECTED -->` markers for that section only if the user explicitly
confirms: "Yes, unlock the [section name] section for editing."

---

## Quality bar

A good CLAUDE.md after this skill runs passes this test:
> *"A new Claude Code instance opening this project cold would have an accurate
> picture of what exists, what the rules are, and what's in progress — with no
> misleading leftovers from earlier plans that were abandoned."*
