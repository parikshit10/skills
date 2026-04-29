# Adding a new skill to this repo

Each skill is self-contained in its own top-level folder. Use this checklist when adding one:

## 1. Folder layout

```
<skill-name>/
├── SKILL.md            # Required
├── references/         # Optional — long-form refs the SKILL.md links to
└── examples/           # Optional — real output runs as fixtures / portfolio
```

## 2. SKILL.md frontmatter

```yaml
---
name: skill-name
description: Use when [specific triggering conditions and symptoms]. Trigger on phrases like "X", "Y", "Z".
---
```

Rules:

- `name` and `description` are the only required fields. Total frontmatter ≤ 1024 chars.
- `name`: kebab-case, letters/numbers/hyphens only, must match the folder name.
- `description`: third person, starts with "Use when...", lists triggering phrases. **Never summarize the workflow** — that causes Claude to follow the description instead of reading the body. Be slightly pushy: list trigger phrases liberally, because Claude tends to under-trigger skills.

## 3. Body of SKILL.md

- Imperative voice ("Run the X scraper", not "You should run...").
- Numbered steps for the core workflow.
- Tables for quick reference (tools, pricing, gotchas).
- Cross-link `references/*` and `examples/*` with explicit "read this when..." pointers.
- Keep under ~500 lines. Push detail into `references/`.

## 4. Commit message convention

`feat: add <skill-name> skill` for new skills. `fix:` / `docs:` / `chore:` for updates. One commit per skill keeps the history scannable.

## 5. Index in README

After adding a skill, add a row to the table in [README.md](./README.md) — name, what it does, and when to use it.

## 6. Test it before pushing

For any non-trivial skill, run it cold in a fresh Claude session before committing. The trigger phrases you wrote in the description should fire it; the workflow should produce something usable. If it doesn't, fix it.
