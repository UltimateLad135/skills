# Delegation

How the orchestrator dispatches, tracks and checks subagents. Every dispatch, in the default flow and in a wrapped skill, follows this file.

## Dispatch

Call `delegate_task` once per subagent:

| Field | Value |
|---|---|
| `task` | The brief (see [Briefs](#briefs)). |
| `title` | `<Role>: <short task name>`, e.g. `Worker: invoice export route`. |
| `role` | Worker `implementation`, advisor `review`, explorer and researcher `research`. |
| `target` | `providerInstanceId`, `model` and `options` from the confirmed team. |
| `interactionMode` | `plan` for advisor, explorer and researcher; omit for workers. |
| `mode` | `async`. Use `wait` only when this turn cannot continue without the result, such as the assessment in step 3. |
| `clientRequestId` | `<run-id>:<role>:<task-id>:<round>`. Stable across retries of the same dispatch, new for every round. |

**Options.** The team stores an effort level per role. Map it onto the option the catalog advertises for that model: the option id differs per provider (`effort`, `reasoningEffort`, `variant`), some models encode effort in the model id itself, and some offer none. Send only options that `orchestrator_capabilities` lists for that model.

**Parallelism.** Dispatch every ready task at once; the host's concurrency is the only limit. File ownership, not headcount, decides what runs in parallel.

## Briefs

Subagents start cold: the brief is everything they know. Every brief opens with the same header, followed by the role's own brief from its definition file:

```
You are the <role> in an orchestrated team. You report only to the orchestrator.
Workspace: <absolute path>. Use absolute paths. Leave every other directory as you found it.
Run folder: <absolute path>. Read the files this brief points to there.
Goal: <the goal card, or the parts this task needs>
Report: <the role's report format>
```

Read-only roles get `Workspace: none (read-only)` and return their report as their final message; the orchestrator saves it to the run folder.

## Waiting

An async child's completion wakes your thread. End your turn while children run rather than polling. Call `task_status` only when you need a result mid-turn, and `task_cancel` for work the plan abandoned.

## Review rounds

Each advisor round is a new `delegate_task` call with its own `clientRequestId`. Its brief carries the original brief, the earlier findings, your responses and any unresolved objections. The `childThreadId` is storage, not a conversation to continue: start every round with a fresh dispatch.

## Guard

Plan mode keeps some providers read-only and not others, so the guard is a check rather than a permission. Record `git status --porcelain` of the integration checkout (the current checkout for a report-only run) before dispatching, and compare it after each child reports:

- **Read-only roles** leave every checkout unchanged.
- **Workers** change only their own worktree.

A difference is a violation. Leave the changes in place, record them in the ledger, and show them to the user before continuing.

## Worktrees

The orchestrator creates every worker's worktree before dispatch, inside the run folder:

```
git worktree add <run-folder>/worktrees/<task-id> -b orchestrate/<slug>--<task-id> <integration tip>
```

Keep task branches beside the integration branch `orchestrate/<slug>`, not beneath it: Git cannot store that branch and `orchestrate/<slug>/<task-id>` together.

A fresh worktree lacks ignored files such as `.env` and installed dependencies. Prepare it the way the repo's own docs describe (README, `AGENTS.md`, `CLAUDE.md`, contributing guide). When they say nothing, ask the user once how to prepare a worktree for this repo, and save the answer in `~/.orchestrate/repos/<repo>.md` for later runs. Keep secrets out of briefs, reports and commits.

Remove a worktree once its branch is merged into the integration branch and its tree is clean. Keep a failed or blocked task's worktree for inspection.

## Ledger

`ledger.md` in the run folder is the run's state, so an interrupted run can resume from it. Record the run's delivery choice, starting branch and commit, push remote, target repository, base branch, review base commit and integration branch where applicable. Keep its PR URL, head commit, draft/ready state and known hosted CI status there too, along with any publishing blocker or reason for skipping the PR. Reconcile these with the host when resuming, and reuse the run's open PR. When a wrapped skill owns the ledger, put this information in that ledger instead.

One row per dispatch:

| Task | Role | Model | State | Worktree / branch | taskId | Commit | Evidence |
|---|---|---|---|---|---|---|---|

States: `pending`, `running`, `awaiting-verification` (committed, required checks not yet passed on the integration branch), `landed`, `failed`, `blocked`. Below the table, record rejected advisor findings with their reasons, guard violations, and user decisions.

## Provider failures

When a child fails on a provider error rather than on the task, retry once. A second failure goes to the user with a suggested replacement model for that role; the team is theirs to change.
