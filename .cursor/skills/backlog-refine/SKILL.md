---
name: backlog-refine
description: >-
  Brainstorm and refine an open backlog card in docs/backlog/open without
  implementing it. Updates the card (and may split it). Grilling is allowed.
  Use when the user runs /backlog-refine, or says refine a backlog item,
  groom a card, brainstorm a parked idea, or sharpen a backlog card before
  work.
disable-model-invocation: true
---

# Backlog refine

Pick an open backlog card and refine it with the user. Do **not** implement code, create a feature branch for coding, or start `/backlog-work`. Do **not** capture unrelated new ideas here — use `/backlog` for that. Do **not** commission or park external research — use `/research`.

Goal: leave a clearer parked card (or split cards) for later. Status stays `open`.

**Canon** is the project's design source of truth (`GDD.md`, `SPEC.md`, `docs/canon.md`, or whatever the repo already uses). Cards are not canon.

## Steps

1. **Resolve the card**
   - If the user already `@`-mentioned a file under `docs/backlog/open/`, or clearly named a slug/title, use that card. Skip the picker.
   - Otherwise list every `*.md` in `docs/backlog/open/` (ignore `.gitkeep`). For each card, read the `#` title and the Size / Layer / Depends on / Ship on / Packages / Parked fields from the table when present.
   - When presenting choices, prefer surfacing Layer and blocked deps. Do not auto-pick.
   - Present a single-select choice with the **AskQuestion** tool when available. One option per card; option label = title, then Size and Layer when present (e.g. `Stale empty-state copy · small · presentation`), optionally packages. Include a freeform escape only if useful.
   - If AskQuestion is unavailable, print a numbered list and wait for a number, slug, or title.
   - Stop until the user selects. Do not brainstorm before a card is chosen.

2. **Load context**
   - Read the chosen card fully.
   - Check relevant canon sections named on the card. Treat the card as intent, not canon; do not invent canon.
   - If Notes (or the card) links `docs/research/`, read those dossiers as **non-canon evidence**. Do not copy hypotheses into **Decisions** unless the user locks them this session.
   - Skim related code or packages only when needed to ground open questions (read-only exploration).

3. **Brainstorm, then write back**
   - Discuss scope, alternatives, non-goals, and ambiguities with the user. Prefer short turns and concrete questions over long essays.
   - **Grilling is allowed.** For a large or fork-heavy card, run a `/grill-me` interview (one hard question at a time) instead of a freeform essay. Same rules: no code, no canon edits, Status stays `open`. Grill questions recommend an option when there is a strong read (`(Recommended)` first); do not auto-pick which card to refine.
   - Update the card in place under `docs/backlog/open/`:
     - Put locks under **Decisions** (add the heading if the card was thin). Do not leave spec in Notes.
     - Notes stay soft links, history, and cautions.
     - Tighten **Why**, **Open questions**, and **Acceptance** from the conversation.
     - Revisit **Size**, **Layer**, **Depends on**, **Ship on**, **Packages**, and **Canon** when the design talk changed them (same allowed values as `/backlog`).
   - Keep Status `open`. Do not move to `done/`.
   - If the card is really multiple concerns, **split**: create additional `open/` cards (same template as `/backlog`) and shrink the original. Wire hard blockers via Depends on; soft links stay in Notes. Assign a shared **Ship on** so `/backlog-work` puts every sibling on one branch:
     - If the original already has a Ship on kebab, inherit it on every new sibling.
     - Else use the original filename slug (without `.md`) so they join `backlog/<original-slug>` if work already started, or create that train later.
     - Do **not** create a git branch from this skill. Stamp the name only.
     - Do not overload Depends on to mean “same feature.”
   - Never invent canon. Park unresolved design in Open questions. Edit the canon doc only if the user explicitly asks.

4. **Stop**
   - Confirm with the card path(s) and a short summary of what changed (and any new split cards, including the shared **Ship on**).
   - If the user then wants to implement, tell them to run `/backlog-work` (do not start implementation in this skill).

## Rules

- Never invent a multi-column triage board; `open/` and `done/` only.
- Backlog cards are not canon until written into the canon doc.
- Do not implement, refactor product code, or open a coding branch.
- Do not commit unless the user explicitly asks.
- Do not use this skill to park brand-new unrelated ideas — redirect to `/backlog`.
- Do not use this skill to commission or park external research — redirect to `/research`.
- Do not use this skill to start build work — redirect to `/backlog-work`.
