---
name: save-session
description: Use when user runs /save-session, asks to "save this session", "log what we did", "capture session context", or at the end of significant sessions with key decisions, debugging breakthroughs, or newly discovered constraints
---

# Save Session

Captures and indexes session knowledge so future sessions quickly orient themselves.

---

## Core Principle

Sessions produce insights, decisions, and constraints that are lost if not recorded. This skill ensures:
- **Consistent structure** for future discovery
- **CLAUDE.md integration** so the session log is always current
- **Automated housekeeping** (trim old entries, handle file conflicts)

---

## When to Use

- User runs `/save-session`
- User says "save the session", "log what we did", "end of session summary", "capture this context"
- Session involved: significant decisions, debugging breakthroughs, architectural changes, newly discovered constraints, or deferred work

**Proactively suggest** when:
- Session ended with deferred work ("we'll do this next time")
- You identified a pattern or constraint that applies to future sessions
- Debugging revealed a root cause that explains past behavior
- An architectural decision was made with trade-offs worth recording
- Multiple "knowledge gaps" were discovered that should block future decisions

---

## Step-by-Step

### 1. Gather Session Context

Scan the full conversation and extract:

| Section | Content |
|---------|---------|
| **Key Insights** | Decisions made, patterns confirmed, techniques that worked |
| **Knowledge Gaps** | Open questions, unverified assumptions, areas needing research |
| **Problems Identified** | Bugs found, design flaws, tech debt, blockers (with file/component) |
| **Solutions Implemented** | What was built/fixed and *why* — enough detail to reconstruct reasoning |
| **Deferred Work** | Explicit TODOs, deliberately left for future sessions |

If a section is empty, mark it `_Nothing to note._` rather than omitting it.

### 2. Confirm Filename

Ask the user to confirm one short question:

```
Session filename: "2026-06-28-session-name-in-snake-case.md" — is this okay?
```

User can confirm or provide a custom name. Validate format: `YYYY-MM-DD-*.md`

### 3. Create `claude-sessions/` Directory

```bash
mkdir -p claude-sessions
```

### 4. Write the Session File

**Path:** `claude-sessions/<filename>`

**Template (use exactly):**

```markdown
# Session Summary: Human-Readable Title

**Date:** YYYY-MM-DD
**Duration / Scope:** One sentence (e.g. "2h, focused on X")

---

## Key Insights

- insight 1
- insight 2

## Knowledge Gaps

- gap 1
- gap 2

## Problems Identified

- problem 1 (file/component reference)

## Solutions Implemented

- solution 1 — include the why, not just the what

## Deferred Work

- [ ] TODO 1
- [ ] TODO 2

---

_Captured by /save-session_
```

### 5. Update CLAUDE.md

**If `## Session Log` does NOT exist:**

Append to the end of CLAUDE.md (create the file if needed):

```markdown
---

## Session Log

> Recent sessions are listed below. Full summaries in `claude-sessions/`.
> At the start of each session, scan this list for relevant prior context.

| Date | Summary | File |
|------|---------|------|
| YYYY-MM-DD | One-line description | [link](claude-sessions/<filename>) |
```

**If `## Session Log` already exists:**

Add a new row (newest at the top):

```markdown
| YYYY-MM-DD | One-line description | [link](claude-sessions/<filename>) |
```

**Trim rule:** Keep max 10 data rows (plus header). If adding a new row would create 11 data rows, delete the oldest data row only — the file in `claude-sessions/` stays untouched in the directory.

### 6. Handle Edge Cases

**File already exists for today:**
Append `-2`, `-3` to filename. Tell the user: "File exists, saving as `-2`."

**CLAUDE.md doesn't exist:**
Create it with minimal header first:
```markdown
# Project Context

_Maintained by Claude Code. Do not edit the Session Log section manually._
```
Then proceed with Step 5.

**Session was very short:**
Still save it. A minimal entry with `_Nothing to note._` is better than lost context.

### 7. Report Completion

Print:
```
✓ Session saved: claude-sessions/<filename>
✓ CLAUDE.md session log updated (N entries)

Deferred work captured:
  - TODO 1
  - TODO 2
```

If there are TODOs, ask: _"Want me to add these to the task tracker?"_

---

## Quality Bar

**Test:** Could a different instance of Claude open this file cold and understand what was decided and why, without re-reading the whole conversation?

Write for that reader.

- ✅ "Fixed off-by-one in `ingest.py:87` — root cause was `[0:n]` slice not including last record during batch flush"
- ❌ "Fixed the bug"
- ✅ "Decided to defer full auth rewrite because legal flagged session token storage — compliance comes first"
- ❌ "Deferred the rewrite"

---

## Red Flags - Stop and Re-Check

- [ ] Filename doesn't follow `YYYY-MM-DD-*.md` format → Fix it
- [ ] CLAUDE.md has no `## Session Log` section and you didn't add one → Add it
- [ ] Session file uses wrong heading levels (H1 instead of H2) → Rewrite
- [ ] Deferred work items aren't checkboxes → Convert to `- [ ]` format
- [ ] User was asked to confirm filename and said no, but you saved anyway → Delete and re-confirm
- [ ] You skipped asking user to confirm filename → Stop, delete the file, and ask the user first
- [ ] Session Log table has more than 10 data rows → Trim it down

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Saving without user confirmation | Always ask the user to confirm filename |
| Omitting empty sections | Mark as `_Nothing to note._` |
| Vague summaries ("fixed the issue") | Include component/file and root cause |
| Forgetting CLAUDE.md update | Session is lost to discovery without it |
| Not trimming old entries | Session Log balloons, making CLAUDE.md unwieldy |
| Using H1 headings in session file | Use H2 so H1 is reserved for title |
