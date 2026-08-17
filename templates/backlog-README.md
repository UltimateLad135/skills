# Product backlog

Capture backlog items from chat without implementing them (feature ideas, improvements, polish, known gaps). Use `/backlog`.

Two capture paths, **one card format**:

| Path | When | What lands in `open/` |
|------|------|------------------------|
| **A — thin** | Spark, refine later | Why + Notes + Open questions. No **Decisions** heading. |
| **B — thick** | Grill first (`/grill-me`), then park | **Decisions** holds the locked v1. Open questions are leftovers. Ready to *start* `/backlog-work` (still plan the build; do not re-grill locks). |

`/backlog-work` does not look for a Path field. It reads the card: Decisions present and specific vs missing.

Optional: `/research` (list + pick, or `@` a card) emits a prompt, then parks a non-canon dossier in `docs/research/` with a Notes link. Still no locks.

Refine a parked card with `/backlog-refine` (list + pick, or `@` a card). Grilling is allowed. Still no implementation.

Start work on an open card with `/backlog-work`.

| Folder | Meaning |
|--------|---------|
| `open/` | Open items; OK to be vague |
| `done/` | Implemented or discarded |

## Size

Every card has a **Size** tag (`small` or `large`). Same workflow either way; Size only tells the agent how hard to plan.

| Size | Meaning |
|------|---------|
| `small` | Clear copy/UI/bug polish; acceptance known |
| `large` | New systems, canon impact, multi-package, or open design questions |

## Layer, Depends on, and Ship on

Light triage fields on every card so a growing backlog stays scannable. **Not** priority scores and **not** extra folders.

| Field | Meaning |
|-------|---------|
| **Layer** | Rough stack order: what kind of work this is |
| **Depends on** | Open card slugs that should come first (hard blockers only) |
| **Ship on** | Shared kebab for split slices that merge together; `—` if this card is its own branch |

### Layer values

| Layer | Prefer when… |
|-------|----------------|
| `foundation` | Shared models, infra, or systems many other cards need |
| `core loop` | The product's defining experience or main user loop |
| `depth` | Richer systems that build on an existing loop |
| `presentation` | UI, visual, copy, polish that does not change domain rules |
| `spice` | Optional extras that need several other systems first |

### Depends on rules

- List **open** kebab slugs only (no `.md`), comma-separated.
- Use `—` when nothing must ship first.
- Soft / thematic links (“pairs with”) stay in **Notes**, not Depends on.
- Dependencies already in `done/` stay in Notes if useful; do not list them as blockers.
- When choosing work, prefer empty Depends on and earlier Layers (`foundation` / `core loop` before `spice`). You may still pick a blocked card deliberately.

### Ship on rules

- `—` (or a missing field on older cards) means a standalone card: branch `backlog/<card-slug>`.
- A kebab slug (no `.md`, no `backlog/` prefix) means this card shares a train with every other open card that has the same Ship on. Branch is `backlog/<ship-on>`. Merge that branch only when **no** open siblings remain.
- Stamp the name when parking or splitting. Do **not** create the git branch until `/backlog-work`.
- Do not use Depends on to mean “same feature.” Parallel siblings can share Ship on with no blocker between them.
- `/backlog-work` on a train branch lists only remaining siblings (plus an escape). On `main`, unfinished trains with an existing `backlog/<ship-on>` branch are offered first.

## Workflow

1. Either:
   - Spark → `/backlog` (thin card), or
   - `/grill-me` → `/backlog` (thick card with **Decisions**).
2. Optional: `/research` (or `@docs/backlog/open/<card>.md`) → copy the prompt to an external model → paste findings back → parks `docs/research/`, Notes link only. Still no locks. Dossiers are not canon.
3. Optional: `/backlog-refine` (or `@docs/backlog/open/<card>.md`) → pick/load → brainstorm or grill → update the card (and split if needed, with a shared **Ship on**). Still parked; no code. Locks go under **Decisions**. If Notes links a research dossier, read it as evidence, not as a spec.
4. Later: `/backlog-work` (or `@docs/backlog/open/<card>.md`) → pick/load the card (on a train branch: remaining siblings only) → choose checkout (**Branch here** or **Worktree**) unless already on that train → plan (light for `small`, thorough for `large`) → implement. Do not re-ask **Decisions**. Ask leftover **Open questions**.
5. When finished, the agent moves the card to `done/`. Update the canon doc if the design became canonical. If Ship on siblings remain, stay on the train; if this was the last (or a standalone), ask about merging.

### Checkout: branch vs worktree

`/backlog-work` asks how to isolate edits before coding:

| Mode | Use when… |
|------|-----------|
| **Branch here** (default) | One agent / one train in this repo folder |
| **Worktree** | A second agent on a *different* train while another is already in flight |

Worktrees live under a sibling `<repo-folder>-worktrees/<branch-kebab>/` folder, on branch `backlog/<ship-on>` or `backlog/<card-slug>`. Install dependencies in that worktree and open it as its own Cursor window. Prefer parallel trains with different **Packages**. Do not put two siblings of the same Ship on in two worktrees. Merge a train to `main` only when its open siblings are gone.

Parked backlog cards are **not** canon.

## Card format

See the `/backlog` skill. One markdown file per item; kebab-case filenames.

| Section | Role |
|---------|------|
| **Why** | One or two sentences of intent |
| **Decisions** | Optional. Locked v1 rules and non-goals. Omit on thin cards. |
| **Notes** | Soft links, history, cautions — not the spec |
| **Open questions** | Forks still open (design or build-time) |
| **Acceptance** | How we’ll know it’s done |

Older `done/` cards may omit **Decisions** or **Ship on**; that is fine (missing Ship on means standalone / `—`).

## Skills

| Skill | Purpose |
|-------|---------|
| `/grill-me` | Interview to lock a v1. No files unless you then `/backlog`. |
| `/research` | Prompt for external research, then park a non-canon dossier in `docs/research/` (Notes link only) |
| `/backlog` | Capture only — write a card (or sibling cards with shared Ship on) to `open/`, do not implement |
| `/backlog-refine` | Pick an open card, brainstorm or grill, update/split (shared Ship on on splits) — still no implement |
| `/backlog-work` | Pick an open card (train-filtered when already on that branch), choose branch or worktree, start work |
| `/pulse` | Read-only architecture pulse. Not a merge gate. |
