---
name: pulse
description: >-
  Read-only architecture pulse: named homes, spaghetti growth, wrong-layer
  leaks, and hotspot files. Reports general findings from git history; open
  backlog cards only mark which rows they would hit. Use when the user runs
  /pulse, or asks for a hotspot check, spaghetti check, file-size health
  check, codebase pulse, or occasional maintainability scan. Not a merge
  review and not Bugbot.
disable-model-invocation: true
---

# Pulse

Occasional, read-only health check of whether the code still has a **named home** for work that does not exist yet.

Do **not** implement, branch, edit the canon doc, or write a backlog card unless the user then asks to park a finding (`/backlog`). This is not a merge gate and not a rewrite brief.

`/grill-me` sharpens design. `/pulse` checks structure. `/backlog` is the save.

**General findings are the scan.** Open cards in `docs/backlog/open/` only mark **Would bite** when they would hit a row. They do not limit what you look at.

## When

After 2–3 large cards in the same area, when a hotspot file grew a lot, or **right before** a large card that will land in a hotspot. Not on every `/backlog-work`.

## Steps

1. **Window, then measure.** Default baseline is the last ~15 **first-parent** commits (`HEAD~15`). Honor an explicit range if the user gives one (`/pulse HEAD~40`). If history is shallower than the window, say so and rank on structure alone.

   In the output, state the range. If first-parent steps ≠ reachable commits (merges), report both. Distinct-feature counting uses diffs and messages in the **reachable** set, not the first-parent step count.

   Discover source files with `git ls-files` (portable). Skip generated output, vendored deps, lockfiles, and build dirs (`node_modules`, `dist`, `build`, `target`, `vendor`, `**/generated/**`). Prefer roots that exist: `packages/`, `apps/`, `src/`, `app/`, `lib/`, `server/`, `backend/`, `frontend/`.

   **Measure committed trees only:** `git show HEAD:<file>` vs `git show <baseline>:<file>` line counts. Do not mix working-tree length with HEAD history. If a candidate is dirty, say so and do not use the dirty length as the growth number.

   Size and a large jump (~300+ lines) only choose where to look. Start with files named in `docs/pulse-hotspots.md` if that file exists, then files that jumped. Skim `docs/backlog/open/` for titles, Size, Layer, and Packages — lens only.

2. **Named-home test, evidenced by history.** A **named home** is a function or module **whose job is that feature**, so further work extends *that* unit. Nested functions count. A sibling module is a stronger seam, not a requirement.

   Not a named home: a new **policy axis** bolted onto an unrelated function, or a generic busy function (`update…`, `sync…`, `handle…`) accumulating unrelated `if (kind)` branches.

   The **symptom must be observed**, not predicted: name the distinct landings inside the window that already arrived as special cases in a busy flow (not in their named home). **Touches ≠ special cases.** **Distinct feature** = one **independent policy axis**. Polish, bugfixes, and follow-ups of the same axis count once.

   Two or more such landings in the same flow is a row; one is not a trend. Hypothetical or open-card work belongs in **Would bite**, never as the sole evidence.

3. **Report at most five rows.** Rank by how badly the next named work would land. Four sharp rows beat five. If nothing meets the bar, verdict `clean` and stop (no table, no split offer).

   **One concern per row.** **Would bite** and **Smallest split** must be the same problem. Two unrelated splits are two rows — or drop the one that fails the two-axis bar. Do not offer two extractions in one smallest-split cell.

   **Verdict is one sentence of advice** (what to do next). Do not put growth numbers, file lists, or evidence in that sentence. Lead with it. Axis: does a seam exist for the named work?

   - **clean** — Do nothing. Do not split; do not park a refactor. Ship the next cards into the named homes already there.
   - **watch** — The next named work can still use an existing seam. Extract that seam when you next touch it; `/backlog` only if you want it queued.
   - **urgent** — Do not pile the named work into the hotspot. Split first, or make the split the first slice of that card.

   Do not hedge `clean` with “park a split if you want one anyway.”

4. **Handoff.** On `clean`, do not mention `/backlog`. On `watch` or `urgent`, if the user wants the split parked, tell them to run `/backlog` (or follow `/backlog` in the same turn if they already said park). Do not invent canon. Suggested splits are options, not Decisions, until grilled or locked. Hint for the card: shared models/infra → Layer `foundation`; UI/render → `presentation`; Size `large` when a package boundary or hotspot contract moves.

## Known hotspots

Do not invent a hotspot list for a foreign repo. If `docs/pulse-hotspots.md` exists, start there, then refresh that list when a hotspot is split.

## Ignore

Generated dumps, dependencies, lockfiles, build output, binary assets, and content-only growth (large constant tables, copy catalogs, generated maps). Those are not spaghetti.

## Flag

- Two or more independent policy axes in the window that landed as special-case branches in an already busy flow (not in their named home)
- Domain rules in the UI layer, or presentation in the domain/core layer
- A near-duplicate helper next to a canonical one
- An existing identity module / thin wrapper next to the thing it wraps

## Do not flag

- Line count alone. A 1k-line file is not a finding.
- A file merely touched by many commits, or a single policy axis in the window (not a trend)
- Work that landed in a named function or module for that feature, including nested helpers that capture factory state
- Cosmetic nits, renames, or “split until files are 400 lines”
- A refactor invented only because an open card exists, or skipped only because none does
- Two unrelated splits packed into one row
- Hedging a `clean` verdict with an optional split

## Remedies (suggest only)

A good module is a **small interface over a large implementation**. Prefer:

- Follow an existing extraction pattern in that area before inventing a new shape
- Extract a phase or layer that already has a name
- Move logic to the package that owns the concept
- Delete a special case rather than wrap it

A nested function that captures a factory is a valid **landing zone**. Extract it to a module only when *that function* already meets the historical bar — capture bags are why the extract is painful, not a reason to flag a named helper. Agent-context cost is a reason to extract a row that already exists, not a reason to create one.

Check your proposed extract against the **deletion test** before suggesting it: if deleting the new module would make the design simpler, do not recommend it.

Do not recommend exploding an orchestrator into many shallow files. Do not recommend refactors that change behavior, canon, package boundaries, or phase order.

## Output

```markdown
## Pulse

**Window:** `HEAD~15..HEAD` (first-parent N; reachable M if different)

**Verdict:** clean — Do nothing. Do not split <module>; ship the next cards into the named homes already there.
```

Name the label (`clean` | `watch` | `urgent`) then the advice sentence. Evidence (growth, names, open cards) comes **after**, never inside the verdict.

Omit the table when the verdict is `clean`. On `watch` / `urgent`, add the table:

```markdown
| Rank | Where | Symptom | Would bite | Smallest split |
|------|-------|---------|------------|----------------|
| 1 | `file` (`symbol` or lines) | observed special-case landings in the window (name the policy axes) | one concrete next work; add `open-slug` only if that same work is an open card | one extract that serves that work |
```

Growth numbers must come from `git show` on the window so they are checkable. Note a dirty worktree separately. Do not dump a file-size leaderboard unless a row needs the number as evidence.

After the verdict (and table if any), one optional **cleared** line: what you opened and found healthy (orchestrators, named nested helpers, module seams, content-only growth).
