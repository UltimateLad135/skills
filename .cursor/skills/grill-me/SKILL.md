---
name: grill-me
description: >-
  Relentless interview to sharpen a plan or design. One hard question at a
  time until a v1 is locked. Use when the user runs /grill-me or /grilling,
  or asks to grill, pressure-test, or sharpen a design before parking or
  building it.
disable-model-invocation: true
---

# Grill me

Sharpen a plan by interview. Do **not** implement code, edit the project canon doc, or write a backlog card unless the user then asks to park it (`/backlog`).

Two ways this shows up in the backlog (same card format either way):

- **Path A** — spark → `/backlog` (thin card, no Decisions) → later `/backlog-refine` (grilling allowed) → `/backlog-work`
- **Path B** — idea → `/grill-me` now → `/backlog` (thick card, Decisions filled) → `/backlog-work` (skip refine)

This skill is the sharpener. `/backlog` is the save.

**Canon** is the project's design source of truth (`GDD.md`, `SPEC.md`, `docs/canon.md`, or whatever the repo already uses). Backlog cards are not canon.

## Steps

1. **Name the job.** In one short beat, restate what is being designed. If two jobs are equally “the point,” say so and make them pick a primary tension.
2. **Ask one hard question per turn.** Prefer numbered mutually exclusive options. Call out what each option kills. Do not dump a full design, a second question, or a spec until they answer.
   - **Recommend when you have a strong read.** Put that option first and append `(Recommended)` on the label (AskQuestion or numbered prose). Still name what it kills. The user can pick against it.
   - If it is a genuine toss-up, do not fake a recommendation.
3. **Lock the pick.** After each answer, state the lock in their words, name the next hole it creates, then ask the next question. Do not silently upgrade their pick.
4. **Stop when they have a v1.** Recap **Decisions** (locks) vs leftover **Open questions**. Do not invent canon.
5. **Handoff.** If they want it parked, tell them to run `/backlog` (or follow `/backlog` in the same turn if they already said park / make a card). `/backlog` must put locks under **Decisions**, not a Q&A transcript. If the v1 is clearly several shippable slices, say so in the recap so `/backlog` writes sibling cards with one shared **Ship on** kebab (still no git branch from this skill).

## Rules

- One question at a time. No essays between answers.
- Default to a recommended option when you have a strong read. Neutral lists are the exception (toss-up only).
- Do not re-open a lock unless the user contradicts it or asks to.
- Do not implement, branch, or edit the canon doc.
- Do not write `docs/backlog/` files from this skill alone — that is `/backlog` or `/backlog-refine`.
- Park unresolved forks in the recap as Open questions, not as fake decisions.
