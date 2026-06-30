---
name: blueprint
description: Use when planning a feature, change, refactor, or bugfix in a codebase and the plan should be reviewed before any code is written — especially after entering plan mode, or when a markdown plan is long and hard to scan. Triggers on "/blueprint", "make a visual plan", "plan this visually", "let me review the plan first".
---

# /blueprint — visual, reviewable plans

## Overview

`/blueprint` renders a plan as a **single self-contained HTML file** the user opens
in a browser: wireframes, a flow diagram, schema and API changes, a file map,
annotated code, and open questions — each section commentable, with one-click
"export all feedback for the agent." It is a **rendering + review layer on top of
normal planning**, not a replacement for it. Do the thinking with the host's
normal planning ability; `/blueprint` makes that thinking legible and reviewable.

**Core principle:** the plan is the approval gate. Surface a blueprint, let the
user comment, apply *all* their feedback at once, then build.

## When to use

- The user runs `/blueprint`, or asks to plan something visually / review before coding.
- Multi-file, UI-heavy, schema/API-touching, ambiguous, or hard-to-reverse work.
- A markdown plan already exists (from plan mode or pasted) and is hard to scan —
  use it as the source and render it; do not re-plan from scratch.

**Skip** for truly trivial work (typo, one-line fix). Just make the change.

## Input — the plan file

`/blueprint` may be invoked with a path to a plan file, e.g. `/blueprint plan.md`.

- **If a path is given**, that file is the **source of truth**. Read it and render it
  — do not re-plan or second-guess its decisions; your job is to make it legible.
- **If no path is given**, look (in order) for `plan.md`, `PLAN.md`, or a plan already
  in context (e.g. from plan mode). Use the first you find.
- **If none exists**, plan natively first (explore the real codebase), then render.

Either way, **resolve every item against the real codebase** before drawing it — a
plan file names intentions; the blocks must name real files, symbols, and endpoints.
Record the plan-file path in the hero `klabel` (e.g. `source · plan.md`) so the
artifact is traceable.

## Workflow

1. **Get the plan (don't reinvent).** Resolve the source per **Input** above — a
   passed plan file, an existing `plan.md`, an in-context plan, or fresh native
   planning. Then explore the real codebase to ground it: read the actual
   files/actions/schema. **Research before drafting** — every block must name real
   files, symbols, columns, and endpoints, even if the plan file was vaguer.
2. **Choose the surface.** UI/product work → include a **Screens** wireframe and
   lead with it. Backend / data / refactor → **no wireframe**; lead with the flow
   diagram, data model, and API. Don't add visual chrome that isn't earned.
3. **Compose the body.** Read `blocks.md` (same directory). Build the hero, then
   one `<section id="...">` per part of the plan, using the block snippets with
   **real content**. Recommended spine: Outcome → (Screens) → Flow → Data model →
   API → Files → Annotated code → Open questions. Include only what applies.
4. **Write the file.**
   - Copy `template.html` to `./blueprints/<slug>.blueprint.html` (create the dir).
   - Set `<body data-plan="<slug>" data-title="<Feature title>">`.
   - Replace `<!-- BLUEPRINT:CONTENT -->` with your hero + sections.
   - Replace `<!-- BLUEPRINT:RAWMD -->` with the plain markdown version of the plan
     (powers the Markdown toggle; this is the "before" the visual plan replaces).
   - Do **not** touch the CSS, the `<script>`, the rail, numbers, or comment
     buttons — those are generated automatically.
5. **Hand off for review.** Open the file in the browser
   (`open ./blueprints/<slug>.blueprint.html` on macOS) and tell the user to
   review, click **✎ comment** on any section, then **✦ Feedback → Copy for the
   agent**. Note which files/areas the work will touch.
6. **Planning is read-only.** Make no source edits until the user approves.
7. **Apply feedback in one pass.** When the user pastes the exported feedback
   (a numbered list of `[§ Section] comment`), apply **every** item — edit the
   `.blueprint.html` sections (and the markdown slot) to match — then re-open it
   and confirm what changed. Only after approval do you write code.

## Plan discipline (keep these from a good plan)

- **Reuse first.** For each part, name what it reuses (existing actions, schema,
  components) before what it adds. The Outcome block exists for this.
- **Decide hard-to-reverse bets in the plan** — wire format, public ids,
  data-model shape, auth/ownership boundaries — even if most of the feature ships later.
- **Open questions get a recommended default.** Anything that would change
  architecture, scope, or data shape goes in the Open questions block with a ★ on
  your recommendation — don't silently decide it, don't ask a separate "looks good?".
- **One concrete example up top.** The hero leads with what the user can actually do.

## Mechanics

- Output: `./blueprints/<slug>.blueprint.html` — fully local, no server, no account.
  Commit it to the repo if you want the plan checked in.
- Comments persist in the browser (`localStorage`, keyed to `data-plan`), so the
  user can close and reopen without losing feedback. The agent receives them via
  the user pasting the exported block — there is no automatic read of the browser.
- `template.html` and `blocks.md` live in this skill directory; read `blocks.md`
  before composing. Do not author blocks from memory.
- **After the feature ships**, the twin skill `/as-built` reads the same plan file
  plus the real code and renders a **planned-vs-built verification**. Keep the plan
  file around (commit it) so the recap has something to check against.

## Common mistakes

- **Placeholder content.** "table_name", "/api/thing" left in. Always use real names.
- **Hand-numbering sections or writing the rail.** The script does both — leave `.num` empty.
- **Wireframes for backend plans.** No canvas for architecture/data-only work.
- **Editing source before approval.** The blueprint is the gate; wait for it.
- **Fixing one comment at a time.** Apply the whole exported list in a single pass.
