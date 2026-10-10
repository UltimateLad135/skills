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

## License

See [LICENSE](LICENSE).
