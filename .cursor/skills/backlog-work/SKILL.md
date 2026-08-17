---
name: backlog-work
description: >-
  Start work on an open backlog card from docs/backlog/open. Lists open
  items and lets the user pick one (AskQuestion when available). On a
  Ship on train branch, lists only remaining siblings. Use when the user
  runs /backlog-work, or says work a backlog item, pick a backlog card,
  start a parked item, or unpark a backlog idea.
disable-model-invocation: true
---

# Backlog work

Pick an open backlog card and start working on it. Do **not** capture new cards here — use `/backlog` for that.

Workflow is the same for every card (pick → load → checkout → plan → implement → move to `done/`). **Size** only changes planning depth — not the steps. **Layer** and **Depends on** inform which card to prefer; they do not change the workflow steps. **Ship on** groups split slices onto one branch (`backlog/<ship-on>`); it is not a folder or a priority score. Checkout is either a feature branch in this repo (default) or an isolated git worktree for a *different* train.

Missing **Ship on** (older cards) means `—`.

**Canon** is the project's design source of truth (`GDD.md`, `SPEC.md`, `docs/canon.md`, or whatever the repo already uses). Cards are intent, not canon.

## Steps

1. **Resolve the card**
   - Read `git branch --show-current` (and whether local `backlog/*` branches exist). Read every `*.md` in `docs/backlog/open/` (ignore `.gitkeep`): `#` title plus Size / Layer / Depends on / **Ship on** / Packages / Parked when present.
   - A **train** is a Ship on kebab that is not `—`. Git branch for a train is always `backlog/<ship-on>`. Standalone cards (`Ship on: —`) still use `backlog/<card-slug>`.
   - **If the current branch is `backlog/<ship-on>` and open cards still share that Ship on:** the picker is **only those remaining cards**. Default is stay on the train. Include one escape, e.g. "Something else (leave this train)". Do not list the rest of the backlog unless they pick the escape.
   - **If on `main` (or any branch that is not this train) and a train still has open cards:** if local branch `backlog/<ship-on>` exists, put **Continue \<train\> (N left)** first, then other open cards. If they pick Continue, next offer only that train’s remaining cards (one pick if N=1). Starting any sibling of a train uses that train’s branch, not a per-card branch.
   - If the user already `@`-mentioned a file under `docs/backlog/open/`, or clearly named a slug/title, use that card. Skip the picker — unless the current branch is an unfinished *other* train; then warn how many siblings remain and ask stay vs switch.
   - Otherwise present a single-select choice with the **AskQuestion** tool when available. One option per card (or Continue-train); option label = title, then Size and Layer when present (e.g. `Stale empty-state copy · small · presentation`), optionally packages and Ship on when not `—`. Include a freeform escape only if useful, e.g. "Something else (I will type it)".
   - When presenting the full backlog (not filtered to a train), prefer surfacing Layer and blocked deps so the user can pick unblocked foundation/core-loop work first. Do not auto-pick or refuse cards with open dependencies — warn briefly if Depends on still points at open cards, then let the user decide.
   - If AskQuestion is unavailable, print a numbered list and wait for a number, slug, or title. Optionally include a markdown link to each card path so the user can open the file; selection still happens by reply, not by clicking the link.
   - Stop until the user selects. Do not start implementing before a card is chosen.

2. **Load context**
   - Read the chosen card fully.
   - Check relevant canon sections named on the card. Treat the card as intent, not canon; do not invent canon. Research dossiers linked from Notes are evidence, not locks.
   - Resolve Size: use the card’s `Size` field (`small` | `large`). If missing, ask once; if still unclear, treat as `large`.
   - Infer **how decided** the card is (not how it was captured — there is no Path field):
     - **Decisions** present and specific → locked spec. Do not re-grill those forks unless the user reopens them.
     - **Decisions** missing/stub and **Open questions** are still product-facing design forks → thin card. Do not invent a design. Ask those forks now, or tell the user to `/backlog-refine` / `/grill-me` first.
     - **Open questions** that are numbers, pipeline details, or migration/breakage may be asked during planning; they do not block starting a thick card.
   - If `Depends on` lists open cards that still exist under `open/`, mention them when planning and ask whether to proceed anyway, design the dependency first, or switch cards.

3. **Choose checkout mode**
   - **Skip this step** when already on the card’s train branch (`backlog/<ship-on>` or `backlog/<card-slug>` for a standalone) and staying on that train. Go to plan/implement.
   - Otherwise ask how to isolate edits before creating a branch or worktree. Prefer **AskQuestion** when available; otherwise a numbered list. Options:
     - **Branch here** (default) — feature branch in this workspace
     - **Worktree** — isolated checkout for a *different* train / parallel agent / another Cursor window
   - If the user already said “worktree”, “parallel”, or “branch here”, skip the ask and use that mode.
   - Do not start plan/implement until checkout mode is chosen (unless they already stated it, or checkout was skipped because they are already on the train).
   - Do **not** open a second worktree for another sibling of a train that is already checked out. Siblings share one branch; work them sequentially in that workspace.

4. **Checkout before edits**
   - Branch name: `backlog/<ship-on>` when Ship on is a kebab; otherwise `backlog/<kebab-slug>` from the card filename (without `.md`).
   - Base: current default branch (`main` unless the repo uses something else). Fetch/update only if needed and safe. If the train branch already exists, check it out (do not recreate from main and drop in-flight work).
   - **Branch here**
     - Create/switch to that branch in this workspace via git if the user agrees, or remind them to create one first if still on `main` with no preference stated.
   - **Worktree**
     - Path: sibling folder `<repo-folder>-worktrees/<branch-kebab>` next to the main repo (e.g. `…/projects/my-app-worktrees/<slug>` when the repo is `…/projects/my-app`). Create the parent directory if missing.
     - Run `git worktree add <path> -b backlog/<branch-kebab> <base>` when the branch does not exist yet; add to the existing train branch if it does (or if the user explicitly wants that).
     - Overlap warn (brief, do not block): list other worktrees (`git worktree list`) and note if another in-flight card likely shares the same **Packages** (or clearly the same hotspot files). Let the user proceed anyway.
     - Remind: install dependencies in the worktree before tests/dev (use the repo’s package manager: `pnpm` / `npm` / `yarn` / `bun` from the lockfile, or the project’s documented install). Open that folder as the Cursor workspace / agent root for this card (this agent should not keep editing the main checkout for this card).
     - If the current session cannot switch roots, stop after creating the worktree and tell the user the path to open; do not implement in the main tree for a worktree-mode card.
   - Do not commit unless the user explicitly asks.
   - When the card is finished and the worktree is no longer needed, the user (or a later request) can remove it with `git worktree remove <path>` — do not remove worktrees unprompted.

5. **Plan, then execute** (always plan; Size sets intensity)
   - Before implementing, make sure you understand what the user wants. Ask remaining **Open questions** (and anything else needed to build). Do **not** re-ask forks already recorded under **Decisions** unless the user wants to reopen them. Do not invent preferences or canon; ask instead.
   - `small`: short plan in chat (confirm acceptance / files / non-goals), then implement. Skip heavy Plan mode unless the user asks or something surprising turns up. Still ask if anything is ambiguous.
   - `large`: thorough Plan mode (or a detailed chat plan) for *how to build*. A thick Decisions section does not skip this — it skips redoing the design interview. Resolve leftover Open questions, check canon gaps, and consider splitting into smaller cards before coding. If you split during work, give new siblings the same **Ship on** (inherit the current train; if the card was standalone, use its filename slug so they join this branch).
   - Implement against the card’s **Decisions** (if any), Why, Acceptance, and canon alignment.
   - Prefer merging/rebasing one parallel *train* at a time if another worktree is in flight (especially shared canon or types). Do not merge a train to `main` while it still has open siblings.
   - When the work for that card is finished, move it from `docs/backlog/open/` to `docs/backlog/done/`, set Status to `done`, and update the canon doc only if the user asked or the design became canonical.
   - **After closeout:** if other open cards still share this Ship on, do **not** suggest merging to `main`. Say how many remain; the next `/backlog-work` on this branch should only offer those. If this was the last sibling (or the card was standalone), say the train is complete and **ask** whether to merge `backlog/<branch>` into `main`. Do not merge unless the user asks.

## Rules

- Never invent a multi-column triage board; `open/` and `done/` only. Layer, Depends on, and Ship on are fields on cards, not extra folders.
- Checkout mode is optional isolation only — do not require worktrees, file-touch lists, or colour coding.
- Backlog cards are not canon until written into the canon doc.
- Do not use this skill to park new ideas — redirect to `/backlog`.
- Do not use this skill only to brainstorm or groom a card without implementing — redirect to `/backlog-refine`.
