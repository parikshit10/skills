---
name: as-built
description: Use after a feature, change, or refactor has been built and you want to verify it against the plan — what was promised vs what actually shipped, what deviated, what's missing. Triggers on "/as-built", "did we build what we planned", "recap the build", "verify against the plan", "planned vs built".
---

# /as-built — planned-vs-built verification

## Overview

`/as-built` is the **after** twin of `/blueprint`. Once a feature ships, it reads the
**plan** and the **real code**, then renders a **single self-contained HTML file**: a
build scorecard, a row-by-row **Planned → As-built** table (each item marked ✓ as
planned / ≈ changed / ✗ not built / ＋ added), the deviations with their reasons, the
schema/API/files *as actually shipped*, and the follow-ups still owed. Every section
is commentable; one click exports a **fix punch-list** for the agent.

**Core principle:** tell the truth about the build. The verdict comes from the code,
not from the plan's optimism. A green row means you verified it in the source.

## When to use

- The user runs `/as-built`, or asks to recap / verify a build against its plan.
- A feature planned with `/blueprint` (or any `plan.md`) has just been implemented.
- Before sign-off, a PR review, or a demo — when "is it actually done?" matters.

**Skip** when nothing was planned to check against, or for a trivial change.

## Input — the plan + the code

`/as-built` may be invoked with a path, e.g. `/as-built plan.md` or
`/as-built ./blueprints/comments.blueprint.html`.

- **The plan is the checklist.** Resolve it (in order): a passed path → the same plan
  file `/blueprint` used → `plan.md`/`PLAN.md` → the in-context plan. Extract every
  intended item (capability, schema change, endpoint, file, open-question decision).
- **The code is the truth.** Inspect what actually shipped: read the changed files,
  run `git diff`/`git log` for the branch, open the real schema/routes/components.
  **Never infer "built" from the plan** — confirm each item in the source.

## Workflow

1. **Build the checklist.** Parse the plan into discrete items. If it came from
   `/blueprint`, its sections (Outcome, Screens, Data model, API, Files, Open
   questions) map directly to rows.
2. **Verify each item against the code.** For every item assign exactly one status:
   `ok` (matches), `dev` (built but differs), `gap` (missing/incomplete), `add`
   (shipped, unplanned). Cite the real file/symbol/route that proves it.
3. **Compose the body.** Read `as-built-blocks.md` (same directory). Recommended
   spine: Hero (verdict) → Build status (scorecard) → **Planned → built** (the core
   table) → Deviations & gaps → Schema/API/Files as shipped → Key code → Follow-ups.
4. **Write the file.**
   - Copy `template.html` to `./as-built/<slug>.asbuilt.html` (create the dir).
   - Set `<body data-plan="<slug>" data-title="<Feature> — as-built">`.
   - Replace `<!-- ASBUILT:CONTENT -->` with your hero + sections.
   - Replace `<!-- ASBUILT:RAWMD -->` with the **original plan markdown** (powers the
     "Original plan" toggle — the thing the build is being checked against).
   - Do **not** touch the CSS, the `<script>`, the rail, numbers, or comment buttons.
5. **Hand off.** Open it (`open ./as-built/<slug>.asbuilt.html`) and tell the user to
   review the verdicts, ✎ comment any row they want fixed or dispute, then **✦ Fixes →
   Copy for the agent**. Point out the gaps and deviations explicitly in chat too.
6. **Close the loop.** When the user pastes the exported fix list, apply **every**
   item, then **re-run `/as-built`** so the scorecard reflects the new reality. Rows
   should turn green or move to Follow-ups — never silently disappear.

## Honesty discipline (what makes this useful)

- **No green without proof.** If you didn't see it in the code, it's not `ok`.
- **A stub is a `gap`, not a `dev`.** "Route exists but returns 501" is not built.
- **Surface every deviation**, even the reasonable ones — give the *why* and note how
  reversible it is. The reviewer decides, not you.
- **Count adds honestly.** Unplanned work that shipped is `add`, not invisible.
- **The verdict must match the rows.** `ship` only if no `gap`; `block` if a required
  item is missing; `caveat` otherwise. The scorecard numbers must reconcile.

## Mechanics

- Output: `./as-built/<slug>.asbuilt.html` — fully local, no server, no account.
- Comments persist in the browser (`localStorage`, keyed to `data-plan`); the agent
  receives them via the user pasting the exported fix list. No automatic read.
- `template.html` and `as-built-blocks.md` live in this skill directory; read the
  blocks file before composing. Do not author blocks from memory.
- Pairs with `/blueprint` (the before twin). Same plan file feeds both.

## Common mistakes

- **Trusting the plan.** "It was planned, so it's built." Verify in the source.
- **Hiding gaps to look finished.** A missing item is the whole point of the recap.
- **Hand-numbering sections or writing the rail.** The script does both — leave `.num` empty.
- **A scorecard that doesn't add up** to the rows. Reconcile counts before shipping.
- **Editing source during the recap.** `/as-built` reports; it doesn't fix. Fixes come
  after the user exports the punch-list.
