# Agent skills

Slash-invoked Cursor skills for parking ideas, grilling a v1, researching evidence, and shipping from a markdown backlog.

They are project-agnostic. Drop them into any repo that wants a thin `docs/backlog/` instead of a kanban board.

## Workflow

```text
spark ──► /backlog (thin card)
              │
idea ──► /grill-me ──► /backlog (thick card, Decisions filled)
              │
              ├─ optional /research   (prompt out → paste in → dossier)
              ├─ optional /backlog-refine
              └─ later /backlog-work  (branch or worktree → implement → done/)

occasional /pulse  (read-only architecture health check)
```

| Skill | Purpose |
|-------|---------|
| [`/grill-me`](.cursor/skills/grill-me/SKILL.md) | One hard question at a time until a v1 is locked. No files. |
| [`/backlog`](.cursor/skills/backlog/SKILL.md) | Park a card in `docs/backlog/open/`. Do not implement. |
| [`/research`](.cursor/skills/research/SKILL.md) | External research prompt, then a non-canon dossier. |
| [`/backlog-refine`](.cursor/skills/backlog-refine/SKILL.md) | Groom or split an open card. Still no code. |
| [`/backlog-work`](.cursor/skills/backlog-work/SKILL.md) | Pick a card, checkout, plan, implement, move to `done/`. |
| [`/pulse`](.cursor/skills/pulse/SKILL.md) | Named-home / spaghetti scan from git history. |

All six set `disable-model-invocation: true`, so they run only when you type `/name` (or attach the skill). They do not auto-fire from chat.

**Canon** means the project's design source of truth — `GDD.md`, `SPEC.md`, `docs/canon.md`, or whatever you already have. Backlog cards and research dossiers are not canon until you write them in.

## Install

From the project that should use these skills:

```bash
npx skills add UltimateLad135/skills --agent cursor
```

Or copy `.cursor/skills/*` into that project's `.cursor/skills/`.

Then copy the [templates](templates/README.md) into the project:

1. `templates/010-backlog-ideas.mdc` → `.cursor/rules/010-backlog-ideas.mdc`
2. `templates/backlog-README.md` → `docs/backlog/README.md`
3. Create `docs/backlog/open/`, `docs/backlog/done/`, and `docs/research/`
4. Optional: `templates/pulse-hotspots.md` → `docs/pulse-hotspots.md`

Restart or refresh Cursor so the skills show up under **Customize → Skills**. Invoke with `/grill-me`, `/backlog`, and so on.

## Card shape

One markdown file per item, kebab-case filename. Size is `small` or `large`. Layer is exactly one of `foundation` | `core loop` | `depth` | `presentation` | `spice`. **Ship on** groups split slices onto one `backlog/<kebab>` branch; `—` means a standalone card.

See `templates/card-template.md` and `templates/backlog-README.md`.

## License

MIT. See [LICENSE](LICENSE).
