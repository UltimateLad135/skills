# Agent skills

## `/orchestrate`

Run a task with a team you assemble yourself, each role on the model you choose.

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
3. **Team**: pick a saved team or preset, adjust any role by picking a suggestion or typing a model, then confirm the goal card and team together.
4. **Explore and research** as needed.
5. **Plan**, reviewed by the advisor.
6. **Build**: workers run in parallel in their own worktrees; the orchestrator merges onto an `orchestrate/<slug>` branch.
7. **Verify** the integrated result with the repo's checks.
8. **Code review** by the advisor; workers fix the accepted findings; the advisor checks the fixes.
9. **Report**. Pushing and PRs happen only when you ask.

Saved teams live in `~/.orchestrate/teams.json`; each run's goal, plan, ledger, reports and worktrees live in `~/.orchestrate/runs/<repo>/<run-id>/`, outside your repo.

## License

See [LICENSE](LICENSE).
