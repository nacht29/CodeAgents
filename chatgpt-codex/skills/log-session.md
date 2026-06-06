---
name: log-session
description: Preserve high-value technical context from the current Codex session into `codex-sessions/` only when the user explicitly types `/log-session`. Use for reusable cross-session logs that capture decisions, debugging steps, changes, constraints, unresolved issues, and project-specific lessons. Do not use automatically during normal coding work, and skip logging when the session has nothing worth preserving.
---

# Log Session

## Purpose

Create a durable Markdown session log for the current repository so future Codex sessions can recover important technical context without rereading the full prior conversation. When a repository-root `AGENTS.md` exists, the log should be discoverable from `AGENTS.md` so later sessions can find it during their normal startup context.

Manual trigger: `/log-session`

## Hard Rules

- Manual-only. Do nothing unless the user's current message explicitly contains `/log-session`.
- Treat `/log-session` as a literal trigger phrase. If this Codex environment does not support native slash-command routing, still require that exact text before acting.
- Never create a session log during normal coding tasks.
- If the session has no meaningful technical decisions, debugging steps, changes, project-specific lessons, or unresolved issues, say there is nothing worth logging and stop.
- Optimize for future Codex sessions, not for diary-style narration.
- Be honest. Do not claim tests passed, bugs were fixed, or work is complete unless that was actually verified in the session.
- Use the current local date when generating the filename.

## What To Log

- Project-specific context that may matter in later sessions.
- Decisions and why they were made.
- Important files, commands, constraints, conventions, and implementation details.
- Bugs, blockers, failed attempts, debugging evidence, and investigation results.
- Changes made and the exact level of verification completed.
- Unresolved follow-ups, risks, or assumptions that future work should revisit.

## What Not To Log

- Generic filler, praise, or conversational chatter.
- Low-value summaries of trivial sessions.
- Repetition of obvious repository facts unless they mattered to the work.
- Claims not supported by the current conversation or local evidence.
- Sensitive secrets that should not be written to disk.

## Workflow

1. Confirm the trigger.
- Proceed only if the user's current message explicitly includes `/log-session`.
- If the message does not include `/log-session`, stop immediately and do not create files.

2. Find the repository root.
- Prefer the current Git worktree root when available.
- If there is no Git root, use the active workspace root.
- Treat the repository-root `AGENTS.md` as the file to update when it exists.

3. Gather evidence from the current session.
- Review the current conversation, the files touched, commands run, outputs observed, and any verification performed.
- Prefer evidence from the current session over assumptions.
- If needed, inspect the local worktree to confirm filenames, diffs, or command details before writing the log.

4. Decide whether the session is worth logging.
- Create a log only when there is durable technical value for future Codex sessions.
- If there is nothing worth preserving, respond with a brief statement that no useful session log should be created.

5. Choose the log filename.
- Create `codex-sessions/` under the repository root if it does not already exist.
- Derive a short session title from the main topic.
- Convert the title to lowercase kebab-case.
- Use the filename format `YYYY-MM-DD-short-kebab-case-title.md`.
- If the same filename already exists, append a numeric suffix such as `-2`, `-3`, and so on.

6. Write the log for future Codex sessions.
- Keep it detailed but practical.
- Capture why decisions were made, not just what changed.
- Mention the important files, commands, conventions, constraints, and verification status.
- Include enough detail that another Codex session can understand the work without needing the original conversation.
- Prefer concrete bullets and evidence over vague prose.
- Use all of the sections below when they add value. If a section has no useful content, write `- None.` or omit it only when omission is clearer than filler.

7. Update `AGENTS.md` when present.
- If `AGENTS.md` exists in the repository root, add or update a section named `## Codex Session Logs`.
- Add a Markdown link in this format:
  `- [2026-01-29 — Debug UI](codex-sessions/2026-01-29-debug-ui.md)`
- Keep links in reverse chronological order where practical.
- Do not duplicate an existing link to the same session log.
- Preserve the rest of `AGENTS.md` exactly as much as possible.

8. If `AGENTS.md` does not exist.
- Do not create `AGENTS.md` just for this skill unless the project already clearly requires it.
- Still create the session log in `codex-sessions/`.
- Mention in the final response that no `AGENTS.md` link was added and that future Codex sessions may not discover the log automatically unless they are pointed to `codex-sessions/`.

9. Report the result.
- State which files were created or modified.
- Include the final session-log path.
- State whether `AGENTS.md` was updated.
- Mention any skipped tests, assumptions, missing verification, or incomplete work captured in the log.

## Log Template

```md
# <Session Title>

Date: YYYY-MM-DD

## Summary
- Concise summary of what happened in the session.

## Key Points
- Important context, decisions, requirements, constraints, and reasoning.

## Issues Encountered
- Bugs, errors, blockers, misunderstandings, failed attempts, or unclear assumptions.

## Debugging / Investigation Steps
- Commands run, files inspected, hypotheses tested, and results observed.

## Changes Made
- Files created, edited, deleted, or refactored.
- Important implementation details.

## New Knowledge / Lessons Learned
- Project-specific knowledge, tool behavior, patterns, or gotchas worth preserving.

## Open Questions / Follow-ups
- Remaining risks, TODOs, decisions to revisit, or suggested next actions.

## Useful References
- Relevant files, commands, docs, links, or related session logs.
```

## Quality Bar

- A future Codex session should be able to read the log and quickly understand the problem, the reasoning, the work completed, the remaining gaps, and where to look next.
- Prefer one strong log over many weak ones.