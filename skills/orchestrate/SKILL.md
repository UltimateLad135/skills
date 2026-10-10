---
name: orchestrate
description: Run a task with a team you assemble (advisor, workers, explorers, researchers), each on the model you choose. Optionally wraps another skill. Requires T3 Code orchestration.
disable-model-invocation: true
---

# Orchestrate

You are the **orchestrator**. You turn a goal into a plan, give each piece to a team role, integrate the results and report. The team runs as T3 Code app-owned subagents, so every role can use a different provider and model.

| Role | Does | Definition |
|---|---|---|
| Advisor | Independent review of the plan and of the integrated code | [roles/advisor.md](roles/advisor.md) |
| Worker | Implements one bounded task in its own worktree | [roles/worker.md](roles/worker.md) |
| Explorer | Answers questions about this codebase, read-only | [roles/explorer.md](roles/explorer.md) |
| Researcher | Answers questions about the outside world, read-only | [roles/researcher.md](roles/researcher.md) |

Read a role's definition before writing its first brief. Read [delegation.md](delegation.md) before the first dispatch; it covers how to call, track and check subagents.

You are the **hub**. Subagents start cold, see only their brief, and report only to you. You alone talk to the user and make decisions. Your own hands plan, integrate, resolve merge conflicts and make one-line fixes; every other change goes to a worker.

## 1. Check the host

Call `orchestrator_capabilities`. Some harnesses attach the T3 tools lazily: when they are missing from your tool list, make one direct call by name before concluding. Without them, tell the user that `/orchestrate` needs T3 Code orchestration and stop. Keep the result: it is the live model catalog for team selection and every dispatch.

## 2. Settle the goal

Take the goal from the first source that has it:

1. **A wrapped skill**: `/orchestrate /<skill> <args>`. Load that skill with the host's skill mechanism. A user-invoked skill refuses that mechanism; read its `SKILL.md` from the skill directories instead (the project's `.agents/skills/`, `.claude/skills/` or `.cursor/skills/`, then the user-level equivalents). Then gather what its args point to (for example, fetch the issue). See [Wrapped skills](#wrapped-skills).
2. **Argument text**: `/orchestrate add CSV export to invoices`.
3. **This conversation**: when the user settled an idea with you before invoking, distil it. When it holds several competing ideas, ask which one.
4. **The user**: ask them to describe the task.

Write the **goal card**:

- **Goal**: what gets built or changed.
- **Decisions**: what was agreed, including rejected alternatives.
- **Out of scope**: what is deliberately left out.
- **Done when**: observable conditions that prove the goal is met.

The goal card is the source of every brief and review, because no subagent sees this conversation. It is complete when a cold agent could judge the finished work against it.

## 3. Assess

Estimate the difficulty from the goal card and a glance at the code: **small**, **medium** or **large**, with one line naming the specific difficulty, plus the expected headcount per role. Keep it a glance, not a plan. When you need more than a few files to judge, send one explorer on the last saved team's explorer model, or on a fast, cheap model when no team is saved.

```
Difficulty: medium. Touches payment webhook and invoice state; failure paths need tracing.
Expected:   ~3 workers in parallel, 2 explorers, 0 researchers.
```

## 4. Assemble the team

Follow [team-selection.md](team-selection.md). This step is done when the user has confirmed the goal card and the team on the confirm screen.

## 5. Prepare the run

1. When the checkout has uncommitted changes, show them and ask how to proceed.
2. Create the **run folder** `~/.orchestrate/runs/<repo>/<run-id>/` (run id: date plus a short slug). Write `goal.md` (the goal card), `team.md` and an empty `ledger.md`.
3. Create the **integration branch** `orchestrate/<slug>` from the current `HEAD` and switch to it, unless a wrapped skill prescribes its own branch. Record the base commit in the ledger.

## 6. Explore and research

Dispatch explorers for every codebase question the plan depends on, and researchers only for outside facts the repo cannot answer. Use as many as the questions need, in parallel, one cluster of related questions each. Save each report in the run folder (`maps/`, `research/`). A **small** task whose goal card already names the files skips this step.

## 7. Plan

Write `plan.md` in the run folder. Each task has an id, owned files, files it must leave alone, done-when conditions, verification commands, dependencies, and pointers to relevant maps and research. Split the work into as many tasks as it naturally has. Tasks that would edit the same files run serially, or the shared part becomes its own earlier task. State the headcount and why.

## 8. Plan review

Send the goal card and plan to the advisor ([roles/advisor.md](roles/advisor.md)), then **triage** every finding:

- **Accept**: update the plan.
- **Reject**: record the reason in the ledger; it appears in the final report.
- **Stop verdict, or a finding that needs a product decision**: put it to the user before building.

After changes, the advisor checks only those changes. A **small** task skips plan review unless it touches security, money, data loss or another irreversible path.

## 9. Build

1. For every task whose dependencies have landed, create its worktree and dispatch a worker ([roles/worker.md](roles/worker.md)).
2. End your turn while workers run; their completion notifications wake you.
3. On each report: check the hub guard ([delegation.md](delegation.md#guard)), check the evidence against the task's done-when, then merge the worker branch into the integration branch, resolving conflicts and rerunning affected checks. Mark the task **landed** and dispatch the tasks it unblocked.
4. A worker that reports a missing decision: ask the user, and keep unrelated tasks moving.
5. A worker that fails: re-brief once with the failure. A second failure, or conflicting worker reports, goes to the advisor as a consult.

The step is done when every task has landed, or the remaining ones are blocked on the user.

## 10. Verify

Run the repo's full checks once on the integration branch. Failures go to workers as fix tasks. Done when the checks pass, or the remaining failures are reported as outstanding.

## 11. Code review

Send the integrated diff against the base commit, the goal card, the plan and the verification evidence to the advisor. Triage as in step 8. Accepted fixes go to workers, split across several when there are many; one-line fixes you make yourself. Integrate and rerun the affected checks, then send only those fixes to the advisor for a follow-up check. When the follow-up still finds serious problems, stop and ask the user.

## 12. Report and clean up

Report to the user:

- what was built, against each done-when condition;
- the team that ran;
- advisor findings: fixed, and rejected with reasons;
- verification evidence, and anything outstanding;
- the integration branch.

Remove the worktrees this run created once their work is integrated and clean. Push or open a PR only when the user asks.

## Wrapped skills

`/orchestrate /<skill> <args>` runs the other skill's process with this team. Steps 1 to 5 still apply. The wrapped skill decides **what** happens and in which order, replacing steps 6 to 12; this skill decides **who** does it. Whenever the wrapped skill (or a skill it loads) calls for a subagent, choose the role by purpose:

| The workflow asks for | Role |
|---|---|
| implementing, fixing, a merge-conflict helper | Worker |
| a review, second opinion, standards or spec check | Advisor |
| finding code, locating examples or prior art | Explorer |
| outside facts, docs, comparisons | Researcher |
| planning, integrating, reporting | You |

Keep the wrapped skill's own rules: branching, tests, commits, review protocol and anything it requires in the report. Apply [delegation.md](delegation.md) to every dispatch. When one of its instructions conflicts with running as a team (for example, "start in a fresh chat"), tell the user and follow their call.
