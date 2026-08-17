---
name: backlog
description: >-
  Capture a backlog item into docs/backlog/open without implementing it
  (feature idea, improvement, polish, or known gap). Use when the user
  runs /backlog, or says park this, backlog this, save this idea, or
  drop an item for later. After a /grill-me session, park locks under
  Decisions (not a transcript).
disable-model-invocation: true
---

# Backlog

Capture the current item as a backlog card. Do **not** implement code, edit the project canon doc, or start the feature.

Same card format always. How full it is depends on the chat:

- **Path A (thin):** a spark. Why + Notes + Open questions. Omit **Decisions** (or leave a stub). Sharpen later with `/backlog-refine`.
- **Path B (thick):** the chat already locked a v1 (usually after `/grill-me`). Fill **Decisions** with those locks, **Open questions** with leftovers, **Acceptance** enough to start work. Skip refine.

Do not add a `Path` or `Grilled` table field. `/backlog-work` infers readiness from whether **Decisions** exist and whether **Open questions** are still product-facing forks.

**Canon** is the project's design source of truth (`GDD.md`, `SPEC.md`, `docs/canon.md`, or whatever the repo already uses). Prefer that path in the **Canon** field. Cards are not canon.

## Steps

1. Extract from the conversation (ask only if missing): short title + one-sentence why.
2. Set **Size** to `small` or `large` (ask only if unclear):
   - `small` — clear copy/UI/bug polish; acceptance already known; hours not days
   - `large` — new systems, canon impact, multi-package work, or open design questions
3. Set **Layer** to exactly one of: `foundation` | `core loop` | `depth` | `presentation` | `spice` (ask only if unclear):
   - `foundation` — shared models, infra, or systems many other cards need
   - `core loop` — the product's defining experience or main user loop
   - `depth` — richer systems that build on an existing loop
   - `presentation` — UI, visual, copy, or polish that does not change domain rules
   - `spice` — optional extras that need several other systems first
4. Set **Depends on** to open-card kebab slugs (no `.md`) that should ship or be designed first, comma-separated; use `—` if none. Soft/thematic links stay in Notes, not here. Done cards stay in Notes only. Depends on is a blocker, not a shared-branch grouping.
5. Set **Ship on**:
   - `—` for a standalone card (default).
   - A shared kebab slug (no `.md`, no `backlog/` prefix) when this turn parks **multiple slices of one v1**, or the conversation locked “these ship together / one branch.”
   - Write the **same** Ship on on every sibling. Prefer a short feature kebab, not the first card’s filename. Ask once if the name is unclear.
   - Do **not** create a git branch. Stamp the name only; `/backlog-work` creates `backlog/<ship-on>` when someone starts the train.
6. Optionally note: relevant canon section(s), and **Packages** (repo areas: apps, packages, folders — or `n/a`).
7. If the conversation has **locked choices**, write them under **Decisions** (rules and non-goals, not Q&A). If it is only a wish, omit Decisions. Notes are history and soft links, not the spec.
8. Create `docs/backlog/open/<kebab-slug>.md` from the template below (one file per slice if splitting). Omit the Decisions heading on thin cards. Create `docs/backlog/open/` (and `done/`) if they do not exist yet.
9. Confirm with the file path(s) and a one-line summary (include Size, Layer, and Ship on when it is not `—`). Stop.

## Rules

- Always write new cards to `open/` (never invent a multi-column triage).
- Always set Size to exactly `small` or `large` (no medium).
- Always set Layer to exactly one allowed value.
- Always set Depends on (`—` or open slugs). Do not invent priority scores or Kanban columns.
- Always set Ship on (`—` or a shared kebab). Do not invent epic folders. Do not `git checkout` from this skill.
- Never invent canon; park open questions instead.
- Do not commit unless the user explicitly asks.
- Do not use this skill to refine an existing open card — redirect to `/backlog-refine`.
- Do not use this skill to park research dumps — redirect to `/research`.
- Do not use this skill to start build work — redirect to `/backlog-work`.
- Do not dump a grilling transcript into Notes.

## Card template

```markdown
# <Title>

| Field | Value |
|-------|-------|
| Status | open |
| Parked | YYYY-MM-DD |
| Size | small / large |
| Layer | foundation / core loop / depth / presentation / spice |
| Depends on | — / open-card-slug, other-slug |
| Ship on | — / feature-kebab |
| Packages | repo areas or n/a |
| Canon | section refs or "TBD — not in canon yet" |

## Why

<one or two sentences>

## Decisions

<optional — omit this heading on thin cards>
<locked v1 rules and non-goals, not a Q&A log>

## Notes

- <soft links, capture history, implementation cautions>

## Open questions

- <ambiguities to resolve before or during implementation>

## Acceptance (optional)

- <how we'll know it's done, if known>
```
