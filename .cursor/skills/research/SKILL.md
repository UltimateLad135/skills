---
name: research
description: >-
  Commission external research for an open backlog card: emit a research
  prompt, then park the paste as a non-canon dossier in docs/research
  with a Notes link on the card. Use when the user runs /research, or
  asks to research a backlog card, write a research prompt, or park
  research findings.
disable-model-invocation: true
---

# Research

Feed an open backlog card with **external evidence**. Two phases, hard stop between them: emit a prompt → user pastes findings → park a dossier.

Do **not** lock design, edit the canon doc, implement, or start `/backlog-refine`. Dossiers are not canon. `/backlog-refine` is what turns hypotheses into **Decisions**.

`/grill-me` sharpens. `/research` gathers evidence. `/backlog-refine` locks. `/backlog` is the save.

**Canon** is the project's design source of truth. Dossiers never become canon by themselves.

## When

A parked card whose **Open questions** would benefit from outside evidence (domain, history, competitors, APIs, regulations — not copy/UI polish). Skip `small` / `presentation` unless the user insists; warn once, then continue.

No open card yet → redirect to `/backlog`. User wants to lock forks → redirect to `/backlog-refine`.

## Detect phase

- Paste, HTML file, or “park this research” **and** a card is known (this chat or `@`) → **Phase 2**.
- Otherwise → **Phase 1**. Stop after the prompt. Do not write files.

## Phase 1 — prompt (no files)

1. **Resolve the card** like `/backlog-refine`: `@` / named slug, else list `docs/backlog/open/*.md` and wait (AskQuestion when available). Do not auto-pick.
2. **Load** the card fully. Skim only the canon sections it names. If Notes already points at `docs/research/`, read that dossier so the prompt does not duplicate it.
3. Infer **in scope** from Why + Decisions; **out of scope** from non-goals and near-miss neighbors. From Open questions, add **1–3 discriminators** — forks outside evidence can settle. Put them in the prompt; do not invent new design questions.
4. Infer a one-line **product framing** from the canon doc or README (what it is, what it is not). Keep it short.
5. Emit a **copy-paste prompt** (one fenced block, ready for Gemini, ChatGPT, or similar). Fill the template below. Then **stop**.

Do not dump Packages, Acceptance, file paths, or implementation details into the prompt — those invite a fake spec.

### Prompt template (fill, then fence)

```text
You are researching facts to inform a design question. Do not design the product.

Product:
<one-line framing from canon or README — what it is and what it is not>

Card intent:
<Why, plus locked Decisions if any — as user/product fantasy, not as engine spec>

Open questions (competing patterns, not a chosen mechanic):
<bullet list from the card>

Discriminators (settle with evidence, not a recommended rule):
<1–3 forks inferred from the OQs>

In scope:
<topics that match the card>

Out of scope:
<near-miss topics; also: source code, formulas-as-canon, implementation roadmaps, schemas, package paths, UI specs>

Constraints:
- A Fact needs a named source (book, article, paper, official docs, archive). Vague attribution is a Gap. If unknown, say "I don't know".
- Do not launder the card: a number or claim that matches a locked Decision is contaminated unless independently sourced. Do not restate product-fantasy locks as evidence.
- Split unlike things. Do not average different systems into one.
- Do not invent types, constants, or equations for the codebase.
- Do not pick a winner. Per open question and discriminator, say what would support A vs B vs C. Hypotheses are optional, labeled HYPOTHESIS, and more than one per question is allowed. Do not close the questions.
- Prefer operational facts: how it works in the world, constraints, failure modes, scarcity, time, cost.

Return markdown with:
1. Facts (named sources)
2. Competing patterns per open question / discriminator (optional HYPOTHESES; not one recommended mechanic)
3. Gaps
```

## Phase 2 — park

1. **Rewrite** the paste into the dossier template. Do not save HTML, charts, TypeScript, or a “Cursor roadmap.” Extract facts and hypotheses; strip the rest. Flag into **Gaps** or **Discarded**: unsourced tables/counts; figures that match a card Decision; unlike things treated as one system; a single recommended mechanic per question (drop winner language; keep competing hooks).
2. Write `docs/research/<card-slug>.md` (kebab, no `backlog/` prefix). Create `docs/research/` if missing. If Notes already links a dossier, **update that file** instead of creating a second one. Also accept an existing `docs/Research/` folder if the repo already uses that spelling — do not create a second research tree.
3. Add one **Notes** bullet on the open card only: `Related research: \`docs/research/<file>.md\` (not canon).` Do not add **Decisions**, do not edit Open questions to “resolve” them from the paste, do not edit the canon doc.
4. Confirm paths. Handoff: when they want to lock forks, `/backlog-refine` and `@` the card; tell it to read the dossier.

If the paste is mostly invented systems (fake formulas, types, migrations), park only the evidential residue and list what was discarded. If nothing evidential remains, do not write a dossier; say so and stop.

### Dossier template

```markdown
# <Card title> — research dossier

| Field | Value |
|-------|-------|
| Status | research — **not canon** until `/backlog-refine` or the canon doc adopts it |
| Backlog | [`docs/backlog/open/<slug>.md`](../backlog/open/<slug>.md) |
| Canon touchpoints | <from the card> |
| In scope | <one line> |
| Out of scope | <one line> |
| Parked | YYYY-MM-DD |
| Source | external paste (unverified) |

## Facts

- <sourced fact>

## Design hypotheses (non-canon)

Each maps to an open question (or discriminator) on the card. Not Decisions. Prefer A vs B hooks; do not pick a winner.

- **OQ:** <question> — A: … / B: … (evidence hooks)

## Gaps

- <what the paste did not establish>

## Discarded from paste

- <one line each: spec, code, formulas, roadmaps stripped>
```

Omit **Discarded from paste** if nothing was stripped. Table/gazetteer form is OK when the card is a content pack; do not force tables onto a spice card.

## Rules

- Slash-only. Do not auto-run because a card was `@`-mentioned.
- Never invent canon. Never copy hypotheses into **Decisions**.
- Do not implement, branch, or commit unless the user asks to commit.
- Do not grill in this skill.
- Do not park research on the backlog card itself (no new sections, no formulas).
- Treat the parked file as an **unverified paste**, even when well formatted.
