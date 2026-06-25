---
name: handoff
description: Use when the user types /handoff, is wrapping up a session, is about to run /clear or start a fresh session, or wants the current work captured so it can be continued later in a new session without losing context.
---

# Handoff

## Overview

Capture everything in the current context into one self-contained note so a brand-new session can pick up exactly where this one left off.

**The next session remembers NOTHING from this conversation. The handoff file is its only memory.** Write every line so it stands on its own without this chat.

## When to use

- The user types `/handoff`
- A session is wrapping up or context is getting full
- Before `/clear`, before closing the terminal, or before starting fresh
- Switching machines or coming back to this work later

## The one rule that matters

**Write for a reader (agent or human) with zero memory of this conversation.**

- Use **absolute paths** — never "the file we edited," "that script," or "the doc from earlier."
- Use **real names, IDs, and URLs** — Google Doc IDs, links, project names, exact commands.
- **State decisions AND their reasons**, so the next session doesn't reopen settled choices.
- If understanding a point would require scrolling up, **write out the actual content** instead of pointing at it.

## Steps

1. Get a timestamp for the filename:
   ```bash
   date +"%Y-%m-%d-%H%M"
   ```
2. Review the **entire conversation** from the first message to now — capture the whole arc, not just the last few exchanges.
3. Make sure the folder exists:
   ```bash
   mkdir -p ~/.claude/handoffs
   ```
4. Fill in the template below and write it (Write tool) to `~/.claude/handoffs/handoff-<timestamp>.md`. Keep a new timestamped file every time — never overwrite an old handoff.
5. Report back: the exact saved path and a paste-ready resume line.

## Handoff template

```markdown
# Handoff — <short title of the work>
*Saved: <full date and time> · Working dir: <absolute path>*

## Goal
What the user is ultimately trying to achieve — their intent, not just the most recent ask.

## Status
One line: where things stand right now. e.g. "3 of 5 scripts drafted; blocked on Drive permissions."

## Done so far
- Concrete completed items, each understandable on its own.

## Key decisions (and why)
- Decision → the reason for it, so it isn't relitigated next session.

## Files & artifacts
- `/absolute/path/to/file` — what it is and its current state
- Google Doc / Sheet / URL / ID — what it is and what's in it

## Context & constraints
- Environment, auth state, tools in play, user preferences that surfaced, gotchas learned.

## Open questions / blockers
- Anything unresolved or waiting on the user.

## Next steps
1. The very next concrete action.
2. Then the next, in order.

## Resume prompt
A line the user can paste straight into a fresh session, e.g.:
"Read ~/.claude/handoffs/handoff-<timestamp>.md and continue from Next steps."
```

Drop any section that genuinely has nothing (e.g. no blockers). Never pad a section to fill it.

## Optional scope argument

If the user adds text after the command (e.g. `/handoff focus on the AWS scripts`), still capture everything, but lead with and expand that area.

## After saving, report

- The full path to the saved file.
- The paste-ready resume prompt line, so it can go straight into the next session.

## Common mistakes

| Mistake | Fix |
| --- | --- |
| Summarizing only the last few messages | Cover the whole conversation arc |
| "the file" / "that doc" / "as discussed" | Absolute paths, real names/IDs, restate the content |
| Vague next steps ("continue the work") | Ordered, concrete, doable actions |
| Padding empty sections | Drop sections that truly have nothing |
| Reusing or overwriting a filename | Always use a fresh `date` timestamp |
| Recording what was done but not why | Capture the reasoning behind each decision |
