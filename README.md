# Skills by Parikshit Pruthi

A growing collection of reusable [Anthropic Skills](https://agentskills.io/specification) I use across Claude Code, Claude Desktop, and Claude.ai. Each skill captures a workflow I run repeatedly so I don't reinvent it every time.

Maintained by [@parikshitpruthi](https://www.instagram.com/parikshitpruthi/) (founder, [Accelerate AI](https://accelerateai.in)).

---

## Skills in this repo

| Skill | What it does | When to use it |
|---|---|---|
| [market-gap-research](./market-gap-research) | Mines X, Reddit, and review sites via Apify to surface product gaps and unmet feature requests, then ships an Instagram carousel / LinkedIn post / research report. | Competitor research, finding SaaS wedges, complaint mining, "what do users wish X did?" |

More skills coming. Each lives in its own top-level folder.

---

## Repo conventions

Every skill follows the same shape so they stay easy to read, install, and remix:

```
<skill-name>/
├── SKILL.md            # Required. YAML frontmatter (name, description) + body.
├── references/         # Optional. Cheatsheets, templates, deep-dives the SKILL.md links to.
└── examples/           # Optional. Real outputs from running the skill — useful as test fixtures.
```

Rules of the road:

- **`SKILL.md` is the contract.** Frontmatter has `name` and `description`. The description is "Use when..." — triggering conditions only, no workflow summary (Claude reads it to decide whether to load the skill).
- **`references/` is for reusable artifacts.** API cheatsheets, HTML templates, prompt libraries — anything `SKILL.md` says "see `references/foo.md` for details."
- **`examples/` is for proof.** A real run of the skill, committed verbatim. Doubles as a test fixture and as a portfolio piece.
- **Skill names are kebab-case verbs or noun phrases** (`market-gap-research`, not `MarketGapResearch` or `gap_research`).
- **One skill per folder, flat at the repo root.** No nesting.

## Installing a skill

### Claude Code

```bash
# Clone once
git clone https://github.com/parikshit10/skills.git ~/parikshit-skills

# Symlink the skills you want into your Claude Code skills dir
ln -s ~/parikshit-skills/market-gap-research ~/.claude/skills/market-gap-research
```

Restart Claude Code. The skill will show up in your skills list and auto-trigger on matching prompts.

### Claude Desktop / Claude.ai

Skills folders here can be uploaded as a project. Drag the skill folder into a Claude Project and the agent will load `SKILL.md` along with its references.

---

## Contributing / using these yourself

These are personal skills tuned for my workflow, but most are general enough to fork. PRs welcome if you spot bugs in the workflow logic or want to add platforms / actor alternatives.

If you fork, swap out the voice/aesthetic notes in each skill (mine are tuned for [@parikshitpruthi](https://www.instagram.com/parikshitpruthi/) and [@parikshitbuilds](https://www.instagram.com/parikshitbuilds/)).

## License

MIT. See [LICENSE](./LICENSE).
