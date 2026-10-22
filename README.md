# Agent skills

## `/orchestrate`

Run a task with a team you assemble yourself, each role on the model you choose, and get a PR to review.

| Role | Does |
|---|---|
| **Orchestrator** | The agent you are talking to. Plans, hands out work, integrates, reports. |
| **Advisor** | Independent review of the plan and of the finished code. Advises; never edits. |
| **Workers** | Implement the tasks, each in its own git worktree, as many in parallel as the plan allows. |
| **Explorers** | Map the codebase, read-only, fast and cheap. |
| **Researchers** | Answer outside questions with dated sources, only when needed. |

Mix providers freely: a Claude orchestrator, a GPT advisor, Grok workers and Gemini explorers is a valid team.

### Requirements

T3 Code with orchestration (app-owned subagents) and at least one provider able to run child tasks. Outside T3 Code the skill says so and stops.

### Install

```bash
npx skills add UltimateLad135/skills --skill orchestrate
```

### Use

```text
/orchestrate                              describe the task when asked
/orchestrate add CSV export to invoices   goal from the argument
/orchestrate /implement 42                run another skill's workflow with your team
```

After brainstorming an idea in a chat, invoke `/orchestrate` in the same chat: it turns the conversation into a goal card instead of asking again.

### How a run goes

1. **Goal card**: goal, decisions, out of scope, done when.
2. **Difficulty**: small, medium or large, with the expected headcount.
3. **Team and delivery**: pick a saved team or preset, adjust any role by picking a suggestion or typing a model, then confirm the goal card, team, starting point and PR target together. You can choose a local-only handoff instead.
4. **Explore and research** as needed.
5. **Plan**, reviewed by the advisor for medium, large or sensitive work; settled small tasks record why review is unnecessary.
6. **Build**: workers run in parallel in their own worktrees; the orchestrator merges onto an `orchestrate/<slug>` branch.
7. **Verify** the integrated result with focused checks for review.
8. **Code review** by the advisor; workers fix the accepted findings; the advisor checks the fixes.
9. **Final gate and PR handoff**: after review fixes are committed, run the complete gate once on an unchanged checkout (through the mandatory pre-push hook when provided), then create or update one PR for you to review. It is ready when the goal conditions, checks and advisor review are satisfied, or a draft with outstanding items listed. The report leads with its link. Merging is yours.

A paused run waits for its unresolved decision or blocker; it publishes a draft during that stop only when you ask. Resuming reuses the run's open PR. Read-only runs stay on the current checkout and create no integration branch. A wrapped skill's existing PR is adopted, and a run with no repository changes produces a report without an empty PR. If publishing fails, the report explains the blocker and preserves the branch and prepared PR description in the run folder's `pr.md`.

Saved teams live in `~/.orchestrate/teams.json`; each run's goal, plan, ledger, reports and worktrees live in `~/.orchestrate/runs/<repo>/<run-id>/`, outside your repo. Invoking `/orchestrate` again with the same goal offers to resume an interrupted run from its ledger.

## `/land`

Land the open PRs from your parallel worktrees in one batch, with one full CI run instead of one per PR.

Merging PRs one by one on GitHub moves the base each time, so every PR that conflicts or falls behind needs a fresh full gate, while PRs that merge cleanly leave combinations nobody tested. `/land` merges every ready PR's head onto one `land/<run-id>` branch, resolves the conflicts, runs the repository's gate once on that combined tree, and fast-forwards the base branch to exactly that commit. GitHub sees each PR's head in the base and marks it merged; the PR branches are never rewritten.

### Requirements

Git and a host CLI (`gh`). Works without T3 Code; with subagents available, judged conflict resolutions get an independent review. The skill reads the gate, merge preconditions and post-merge duties from the repository's `AGENTS.md`, `CLAUDE.md` and CI docs.

### Install

```bash
npx skills add UltimateLad135/skills --skill land
```

### Use

```text
/land               every open PR that targets the default branch
/land 12 15 18      only these PRs
/land --plan        show the batch, order and expected conflicts; change nothing
```

### How a run goes

1. **Rules**: the base branch, the gate and whether a pre-push hook runs it, merge preconditions such as preview deployments, post-merge duties, branch protection.
2. **Batch**: eligible PRs (not draft, no requested changes, required checks green, nothing outstanding) in a bisect-friendly order; everything else is deferred with a reason.
3. **Merge** each PR head with `--no-ff` in a dedicated worktree. Mechanical conflicts are resolved, lockfiles and generated files regenerated; a conflict that needs judgment is resolved only when both intents are clear, otherwise that PR is deferred and the rest continue.
4. **Gate once** on the combined commit. With a pre-push hook that reuses a verified tree, pushing the landing branch runs the gate and builds previews, and the push to the base then costs seconds.
5. **On failure**: defer the PR that caused it and rebuild, or bisect when the cause is unclear.
6. **Land** with a fast-forward push to the base, or one combined merge-commit PR when protection forbids direct pushes. Never `--no-verify`, never force-push.
7. **Close out**: confirm each PR shows merged, close issues it names, run post-merge duties the docs assign to agents, and report what landed, what was deferred and why, every conflict resolution and the gate evidence.

Each run keeps its ledger and landing worktree in `~/.land/runs/<repo>/<run-id>/`, so an interrupted run can resume.

## License

See [LICENSE](LICENSE).
