---
name: obsidian-manager
description: >
  Manages an Obsidian vault's graph structure using a strict hub-and-spoke hierarchy so
  that no single node balloons in the graph view. Use this skill whenever the user runs
  /obsidian-manager, asks to create a node, refactor nodes, audit the vault graph,
  or mentions Obsidian node hierarchy, graph layout, hub nodes, or leaf nodes.
  Also trigger when the user provides a path like "Studies > Git > Merge Conflicts"
  implying a parent-child node relationship in an Obsidian vault.
---

# Obsidian Manager

## Purpose

Obsidian's graph view sizes nodes by their number of incoming links. Linking every note
directly to one central concept (e.g. "Git") causes that node to balloon visually.
This skill enforces a **root → hub → leaf** hierarchy so visual weight stays intentional
and the graph remains readable.

The vault is opened as the current working directory in Claude Code. All file paths
are relative to that root.

---

## The Hierarchy Model

```
Layer 1   Root          e.g. Studies
  └── Layer 2   Hub      e.g. Git
        └── Layer 3   (sub-hub / leaf)   e.g. Merge Conflicts
              └── Layer 4 ...
```

**Layer = depth from the root.** The root is always Layer 1. Each step down increments
the layer by one. The hierarchy can go arbitrarily deep; "Hub" and "Leaf" are just
descriptive positions (a node with children is a hub, a node with none is a leaf).

**Rules:**
- A node links only to its direct children one layer below. It never links upward.
- A **Leaf** (no children) has no outgoing links to other notes in the hierarchy.
- Links are **one-directional** and always flow downward: Layer N → Layer N+1.
- No cross-links between siblings at any level unless explicitly requested.

---

## Layer Coloring

**Only roots and hubs are color-coded.** Leaves (nodes with no children) use Obsidian's
default node color. This keeps the graph clean — color signals structural level, not
every individual note.

Coloring is driven by two things working together:

1. **A layer tag on root and hub notes only** — `#layer-1`, `#layer-2`, … added to
   the note's frontmatter. Leaf notes get **no layer tag**.
2. **Color groups in `.obsidian/graph.json`** — one group per hub/root layer,
   matching `tag:#layer-N` to a standard color.

### Standard palette (layers 1–10)

Cool-toned spectrum running from deep teal at the root through blue, indigo, and violet
at depth. The decimal `rgb` values are what `.obsidian/graph.json` expects.

To convert any hex to the decimal Obsidian wants: `rgb = R*65536 + G*256 + B`.

| Layer | Name         | Hex       | Obsidian rgb (decimal) |
|-------|--------------|-----------|------------------------|
| 1     | Deep teal    | `#00736B` | 29547                  |
| 2     | Teal         | `#0B8FAC` | 756652                 |
| 3     | Sky blue     | `#1E96D4` | 2069204                |
| 4     | Cornflower   | `#4A80C4` | 4882628                |
| 5     | Slate blue   | `#5B6DC8` | 5990856                |
| 6     | Blue-violet  | `#6A5ACD` | 6970061                |
| 7     | Indigo       | `#5C4F9E` | 6049694                |
| 8     | Deep indigo  | `#4B3F8A` | 4145034                |
| 9     | Mauve        | `#7B5EA7` | 8085159                |
| 10    | Dusty violet | `#8B7AB8` | 9140664                |

### `.obsidian/graph.json` color group format

Read the existing file first. Merge only the `colorGroups` key — never overwrite
other keys. Only add groups for layers that actually have root/hub notes in the vault.

```json
{
  "colorGroups": [
    { "query": "tag:#layer-1", "color": { "a": 1, "rgb": 29547 } },
    { "query": "tag:#layer-2", "color": { "a": 1, "rgb": 756652 } }
  ]
}
```

### Recolor on overflow (depth > 10)

If hub depth grows past layer 10:

1. Generate a fresh cool-toned palette with one distinct color per layer —
   evenly-spaced hues between 160°–280° HSL, consistent saturation and lightness.
2. Recolor **all** layers, not just the new ones.
3. Rewrite the `colorGroups` array and update `.obsidian/graph-settings.md`.
4. Report that a recolor happened and show the new palette.

---

## Settings File

All node role designations are persisted in `.obsidian/graph-settings.md`.
Read this file at the start of every command. Create it if it doesn't exist.
**Never create `_graph-settings.md` at the vault root** — always use `.obsidian/graph-settings.md`.

### Format

```markdown
# Obsidian Graph Settings

## Nodes
- Studies (layer: 1, role: root)
- Work (layer: 1, role: root)
- Git (layer: 2, role: hub, parent: Studies)
- Interview (layer: 2, role: hub, parent: Work)
- Merge Conflicts (role: leaf, parent: Git)
- Rebase vs Reset (role: leaf, parent: Git)
```

Roots and hubs record their layer number. Leaves record only their role and parent —
they have no layer tag and are not color-coded. Always update this file after any
create or refactor operation.

---

## Commands

### `/obsidian-manager create node <path>`

Creates one or more notes following the hierarchy path provided.

**Path syntax:** `A > B > C`
- The number of segments determines the level of each node.
- A 3-part path like `Studies > Git > Merge Conflicts` means Root > Hub > Leaf.
- A 2-part path like `Git > Merge Conflicts` is resolved using settings (see Inference Rules below).

#### Step-by-step

1. Read `.obsidian/graph-settings.md` and `.obsidian/graph.json`.
2. Parse the path into segments.
3. Resolve roles and **layer depth** for each segment using the Inference Rules.
4. For each segment that doesn't already have a note file, create it.
5. Wire links: each note links to the note directly below it (parent → child).
6. **Apply the layer tag** (`#layer-N`) to root and hub notes only. Leaves get no tag.
7. **Update `.obsidian/graph.json`** — ensure a color group exists for every root/hub layer now present.
8. Update `.obsidian/graph-settings.md` with new role/layer assignments.
9. If the new hub depth exceeds 10 layers, run the recolor process (see Recolor on overflow).
10. Report what was created, what already existed, color changes, and any assumptions made.

#### Inference Rules

Apply these in order when a segment's role is ambiguous:

1. **Already in settings** → use the stored role. No change needed.
2. **3-part path** → first segment is Root, second is Hub, third is Leaf.
3. **2-part path, first segment is a known Hub** → treat as Hub > Leaf.
4. **2-part path, first segment is a known Root** → treat as Root > Hub (no Leaf created).
5. **2-part path, first segment role is unknown** → FLAG. Ask the user:
   "I don't recognise `[segment]` as a Root or Hub yet. Should it be a Root (top-level topic)
   or a Hub under an existing Root? If it's a Hub, which Root does it belong to?"
6. **Segment already exists at a different level** → FLAG before proceeding (see Flagging).

#### Note file format

**Root or Hub note** (gets a layer tag):
```markdown
---
tags:
  - layer-2
---
# [Note Title]

## Links
- [[Child Note 1]]
- [[Child Note 2]]
```

**Leaf note** (no tag, no Links section):
```markdown
# [Note Title]
```

---

### `/obsidian-manager refactor <description>`

Audits and restructures existing notes to conform to the hierarchy.

**Common triggers:**
- "Git is too big, Studies should be the main node"
- "Reorganise the Git cluster"
- "Audit the vault"

#### Step-by-step

1. Read `.obsidian/graph-settings.md`.
2. Scan all `.md` files in the vault (excluding `.obsidian/` directory).
3. Parse `[[wikilinks]]` in each file to build a live link map.
4. Compare the live link map against the settings to find violations:
   - Root linking directly to a Leaf (skipping Hub level)
   - Hub linking up to its Root (upward link)
   - A node with many incoming links that isn't designated as a Hub
   - A node designated as Root but receiving direct links from Leaves
5. Propose a refactor plan in plain language before touching any files.
6. If the scope is large or ambiguous, FLAG (see Flagging) and wait for confirmation.
7. On approval, apply changes: rewrite link sections, **re-tag roots/hubs whose layer
   changed** (`#layer-N`), **strip tags from any node that becomes a leaf**, update
   `.obsidian/graph.json` color groups, and update `.obsidian/graph-settings.md`.
8. If the deepest hub layer now exceeds 10, run the recolor process.
9. Report a summary of every file touched and any color changes.

#### Refactor scope guidance

- **Small refactor (≤5 files changed):** Apply immediately, then report.
- **Medium refactor (6–15 files):** Show the plan, ask "Proceed?", then apply.
- **Large refactor (>15 files) or ambiguous moves:** Always FLAG. Do not proceed without explicit user approval.

---

### `/obsidian-manager audit`

Read-only check. Scans the vault and reports violations without making changes.

Output format:

```
## Vault Audit

### Hierarchy violations
- `Merge Conflicts.md` links to `Git.md` (upward link — leaves should not link up)
- `Studies.md` links directly to `Rebase.md` (root skipping hub level)

### Layer / color issues
- `Git.md` tagged `#layer-3` but should be `#layer-2` (hub under Studies)
- `Merge Conflicts.md` has a `#layer-3` tag but is a leaf — tag should be removed
- `.obsidian/graph.json` missing a color group for layer 4

### Nodes with unresolved roles
- `Cloud Space.md` — not in settings, has 3 incoming links

### Suggested actions
- Move direct links from `Studies.md` to the appropriate hub
- Re-tag `Git.md` to `#layer-2`
- Remove layer tag from `Merge Conflicts.md`
- Add the layer-4 color group to `.obsidian/graph.json`
- Register `Cloud Space.md` with a layer and role
```

---

## Flagging

When something is ambiguous or the impact is large, stop and present a Flag block:

```
⚑ FLAG — [brief title]

Situation: [what was found]
Options:
  A. [option A]
  B. [option B]
  C. Let me decide manually

Waiting for your instruction before proceeding.
```

Situations that always require a Flag:
- A node's role cannot be inferred from settings or path structure
- A refactor would move or rewrite more than 15 files
- A new path conflicts with an existing role in settings
  (e.g. user says `Git > Studies > Merge Conflicts` but Studies is already a Root)
- Two plausible parent Hubs exist for a Leaf and neither is clearly preferred

---

## File Naming Convention

- Use the note title exactly as provided, with capitalisation preserved.
- File name = `[Title].md` (spaces allowed; Obsidian handles them fine).
- Hub notes may optionally be prefixed with their Root for disambiguation
  if the vault already contains a same-named note at a different level —
  but only do this if there's an actual name collision.

---

## Worked Examples

### Example 1 — Full 3-part path, nothing exists yet

Command: `/obsidian-manager create node Studies > Git > Merge Conflicts`

Steps:
1. Read `.obsidian/graph-settings.md`. `Studies` is a Root. `Git` and `Merge Conflicts` are unknown.
2. 3-part path → Studies = Root (layer 1), Git = Hub (layer 2), Merge Conflicts = Leaf.
3. `Studies.md` exists. Add `[[Git]]` to its Links section. Ensure it has `#layer-1` tag.
4. Create `Git.md` with `#layer-2` tag and `[[Merge Conflicts]]` in its Links section.
5. Create `Merge Conflicts.md` with no tag and no Links section.
6. Register Git (layer: 2, hub, parent: Studies) and Merge Conflicts (leaf, parent: Git) in settings.
7. Ensure `.obsidian/graph.json` has color groups for layer-1 and layer-2.

Report:
```
✓ Studies.md — updated (added link to Git, confirmed layer-1 tag)
✓ Git.md — created (hub, layer 2, #00736B → teal)
✓ Merge Conflicts.md — created (leaf, no color)
.obsidian/graph-settings.md updated.
.obsidian/graph.json updated (2 color groups).
```

---

### Example 2 — 2-part path, Hub already known

Command: `/obsidian-manager create node Git > Rebase vs Reset`

Steps:
1. Read `.obsidian/graph-settings.md`. `Git` is a Hub (layer: 2, parent: Studies).
2. 2-part path, first segment is a known Hub → Hub > Leaf.
3. Add `[[Rebase vs Reset]]` to `Git.md`'s Links section.
4. Create `Rebase vs Reset.md` with no tag and no Links section.
5. Register Rebase vs Reset (leaf, parent: Git) in settings.

Report:
```
✓ Git.md — updated (added link to Rebase vs Reset)
✓ Rebase vs Reset.md — created (leaf, no color)
.obsidian/graph-settings.md updated.
```

---

### Example 3 — Ambiguous 2-part path

Command: `/obsidian-manager create node Cloud Space > Tranglo`

Steps:
1. Read settings. `Cloud Space` is not in settings. Role unknown.
2. Cannot infer from a 2-part path with an unknown first segment.

Flag:
```
⚑ FLAG — Unknown role for "Cloud Space"

Situation: `Cloud Space` isn't registered in settings yet.
Options:
  A. Make Cloud Space a Root (top-level topic)
  B. Make Cloud Space a Hub — if so, which Root does it belong to?
  C. Let me decide manually

Waiting for your instruction before proceeding.
```

---

### Example 4 — Refactor: Git is ballooned

Command: `/obsidian-manager refactor Git is too big, Studies should be the main node`

Steps:
1. Scan vault. Find all files linking to `Git.md`. Suppose there are 9 such links.
2. Confirm `Studies` is a Root. `Git` should be a Hub under Studies, not a Root.
3. Plan: ensure `Studies.md` links to `Git.md`; ensure all leaves link only from
   `Git.md`, not from `Studies.md`. Tag `Studies.md` with `#layer-1`, `Git.md` with
   `#layer-2`. Strip any layer tags from leaf notes.
4. 9 files affected → medium refactor. Show plan and ask "Proceed?".
5. On approval, rewrite link sections accordingly. Update settings and graph.json.

---

## Summary Checklist (run mentally before every operation)

- [ ] Did I read `.obsidian/graph-settings.md` and `.obsidian/graph.json` first?
- [ ] Did I resolve every segment's role AND layer depth before creating or editing files?
- [ ] Are all links one-directional (parent → child only)?
- [ ] Did I apply `#layer-N` tags to roots and hubs only — and leave leaf notes untagged?
- [ ] Did I ensure a color group exists in graph.json for every root/hub layer present?
- [ ] If hub depth exceeded 10, did I regenerate and apply a fresh cool-toned palette?
- [ ] Did I flag anything ambiguous before acting?
- [ ] Did I update both `.obsidian/graph-settings.md` and `.obsidian/graph.json`?
- [ ] Did I report every file created or changed, plus any color changes?
