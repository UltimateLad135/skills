---
name: orchestrate
description: Run a task with a team you assemble (advisor, workers, explorers, researchers), each on the model you choose, and hand off repository changes as a PR for review. Optionally wraps another skill. Requires T3 Code orchestration.
disable-model-invocation: true
---

# Orchestrate

You are the **orchestrator**. You turn a goal into a plan, give each piece to a team role, integrate the results and hand off a PR for the user to review. The team runs as T3 Code app-owned subagents, so every role can use a different provider and model. Merging the PR is the user's decision.

| Role       | Does                                                      | Definition                                 |
| ---------- | --------------------------------------------------------- | ------------------------------------------ |
| Advisor    | Independent review of the plan and of the integrated code | [roles/advisor.md](roles/advisor.md)       |
| Worker     | Implements one bounded task in its own worktree           | [roles/worker.md](roles/worker.md)         |
| Explorer   | Answers questions about this codebase, read-only          | [roles/explorer.md](roles/explorer.md)     |
| Researcher | Answers questions about the outside world, read-only      | [roles/researcher.md](roles/researcher.md) |

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

Follow [team-selection.md](team-selection.md). This step is done when the user has confirmed the goal card, team and delivery on the confirm screen. A run that changes the repository ends with a PR by default; the user can choose a local-only handoff.

## 5. Prepare the run

1. Use the new-run or resume choice settled before confirmation. A resumed run reuses its run folder, branch, ledger and open PR. Before dispatching anything, reconcile the ledger: call `task_status` for every `running` row, read the commits on each task branch and the integration branch, and check any recorded PR on its host. Link an existing PR to this T3 thread when starting work on it. Where the ledger and the commits or host state disagree, the observed state wins. Then continue at the first unfinished step without recreating the run's setup.
2. When the checkout has uncommitted changes, show them and ask how to proceed.
3. Create the **run folder** `~/.orchestrate/runs/<repo>/<run-id>/` (run id: date plus a short slug). Write `goal.md` (the goal card), `team.md` and an empty `ledger.md`.
4. Record the confirmed delivery and starting point, plus the push remote, target repository and base branch when publishing. For a run expected to change the repository, create the **integration branch** `orchestrate/<slug>` from the confirmed starting point, unless a wrapped skill prescribes another branch, and switch to it. Preserve the current `HEAD` unless a different start was agreed. Record the starting commit separately from the **base commit** used for review: the refreshed target-base tip for PR delivery, or the starting commit for local-only delivery. This lets the advisor review any earlier commits explicitly included in the PR. If the target cannot be refreshed, use its last known tip or the starting commit for local review and record missing PR-base validation as a publishing blocker. A report-only run keeps the current checkout and creates no integration branch; if it turns out to need edits, settle the changed goal and delivery before dispatching a worker.

## 6. Explore and research

Dispatch explorers for every codebase question the plan depends on, and researchers only for outside facts the repo cannot answer. Use as many as the questions need, in parallel, one cluster of related questions each. Save each report in the run folder (`maps/`, `research/`). A **small** task whose goal card already names the files skips this step.

## 7. Plan

Write `plan.md` in the run folder. Each task has an id, owned files, files it must leave alone, shared resources, done-when conditions, verification commands, dependencies, and pointers to relevant maps and research. Split the work into as many tasks as it naturally has, and map every done-when on the goal card to at least one task. State the headcount and why. Name the final gate command and its owner, schedule it after review fixes, and distinguish it from workers' focused checks. Protect the active integration checkout from host cleanup; record any worktree lock this run adds so it can be released at handoff.

Worktrees isolate files, not everything parallel tasks can collide on. Settle each collision in the plan:

- **Shared files**, including one registry, lockfile or generated file that several tasks append to or regenerate: run those tasks serially, make the shared part its own earlier task, or write the exact interface (types, signatures, file ownership) into the plan so both tasks build against it in parallel.
- **Ordered artifacts**, such as migration numbers: assign them in the plan.
- **Shared resources** that verification uses, such as a local database, containers or fixed ports: you own them. Tasks that use the same resource run serially, or their workers leave that check to you and you run it during integration.

## 8. Plan review

Send the goal card and plan to the advisor ([roles/advisor.md](roles/advisor.md)), then **triage** every finding:

- **Accept**: update the plan.
- **Reject**: record the reason in the ledger; it appears in the final report.
- **Stop verdict, or a finding that needs a product decision**: put it to the user before building.

After changes, the advisor checks only those changes. Review **medium**, **large** and sensitive tasks before building. A **small** task skips plan review when its scope and verification are settled; record the reason in the plan. Sensitive work includes security, money behavior, data loss, irreversible paths, and changes to the meaning of prices, tax statements or contractual terms. Restating already-approved facts without changing their meaning can use the small-task exception.

## 9. Build

1. For every task whose dependencies have landed, create its worktree and dispatch a worker ([roles/worker.md](roles/worker.md)).
2. End your turn while workers run; their completion notifications wake you.
3. On each report: check the hub guard ([delegation.md](delegation.md#guard)), check the evidence against the task's done-when, then merge the worker branch into the integration branch, resolving conflicts and rerunning affected checks. Run every required check the worker skipped or left to you there as well. Mark the task **landed** once its required checks pass on the integration branch, and dispatch the tasks it unblocked. Until then it stays `awaiting-verification`, and its dependents wait.
4. A worker that reports a missing decision: ask the user, and keep unrelated tasks moving.
5. A worker that fails: re-brief once with the failure. A second failure, or conflicting worker reports, goes to the advisor as a consult. A task that stays failed or blocked holds its dependents; unrelated tasks keep moving.
6. When tasks remain but none is running and none can start, the build has **stalled**: report each waiting task with the dependency, decision or cycle it waits on, and keep the ledger for a resumed run.

The step is done when every task has landed. A stalled run stops here and resumes when its blockers are resolved.

## 10. Verify

Run the affected checks on the integration branch and retain evidence for code review. Workers supply focused evidence; the hub owns the final gate. Failures go to workers as fix tasks. Done when affected checks pass, or the remaining failures are reported as outstanding. The complete repository gate follows review fixes in step 12; a required pre-push gate supplies that run rather than duplicating it here.

## 11. Code review

Send the integrated diff against the base commit, the goal card, the plan and the focused verification evidence to the advisor, identifying the pending final gate. Triage as in step 8. Accepted fixes go to workers, split across several when there are many; one-line fixes you make yourself. Integrate and rerun the affected checks, then send only those fixes to the advisor for a follow-up check. When the follow-up still finds serious problems, stop and ask the user.

## 12. Hand off as a PR

Enter this step after verification and advisor review, or the wrapped workflow's required checks and reviews. A run awaiting a user decision at a stop verdict, missing decision or stalled build stays stopped; create a draft during that stop only if the user asks. A final handoff with outstanding checks or other unfinished work is a draft, not a claim that the goal is complete.

After review fixes are committed, run the complete repository gate once on a stable checkout. For PR delivery, use the mandatory pre-push hook when it provides that gate; otherwise run the prescribed gate before pushing. For local-only delivery, run it directly. Finish checkout writers and competing heavy local checks first; remote read-only advice need not block it. Keep the checkout unchanged until the gate ends. Record the reviewed and verified commits and report phase/elapsed progress during long checks or shared-resource waits. A changed checkout invalidates the run. Gate failures go back through affected fixes and review before a fresh final gate; one successful gate is the target, not a limit on repairing failures.

For a run with repository changes and confirmed PR delivery:

1. Check that the intended changes are committed. Refresh the target base and inspect the prospective PR's diff and commits against it. The diff must match the work the advisor reviewed and contain no unrelated changes. A changed diff goes through the affected checks and advisor review before being marked ready, within the normal review-round limit.
2. Look up the run's PR in its ledger, then on the host by head branch and repository if necessary. Reuse an open PR only when its head and target match the confirmed delivery. If an adopted or recorded PR differs, report the mismatch and settle the delivery using existing user decisions, asking only when those do not resolve it; never create a second PR for the same work. If a recorded PR is closed or merged, report that state and settle whether a new run is needed. Push only the integration branch to the confirmed push remote, using a normal fast-forward push. A remote branch with unrelated or divergent history needs resolution before publishing; do not force-push it.
3. Prepare the PR title and body in `pr.md` in the run folder, following the repository's PR template. Describe the resulting behavior, goal conditions met or outstanding, verification evidence, material limitations, and rejected advisor findings with their reasons. Keep secrets and private run-folder paths out of the description. Create the PR or update the existing one using the draft/ready conditions below from the outset. Immediately call `link_pull_request` after creating a PR, or when starting work on an existing one, and record its URL in the run's ledger.
4. Make the PR ready for review only when every goal condition is met, required local checks pass, required reviews are complete and no critical or major review finding remains unresolved. Otherwise create a draft or convert an existing ready PR back to draft, listing the outstanding items; settle any conflict with a wrapped skill's publishing rules rather than silently leaving it ready. On resume, update the same PR's description and promote it when those conditions are met. Report hosted CI separately: pending or unavailable is not passed, and a known failing required check or known merge conflict keeps the PR in draft. Inspect its available status once at handoff; monitor it only when the user asks.
5. Record the PR's head commit, draft/ready state and known CI status in the ledger. Call `list_thread_pull_requests` and link any PR from this run that is missing before reporting the handoff. Leave merging to the user.

When the user chose local-only delivery or the run produced no repository changes, skip publishing and say why. If publishing is unavailable or fails, keep the branch, save the prepared PR title and body in the run folder's `pr.md`, record the failed handoff and explain the concrete blocker with a pointer to that file in the report. Use an available host tool or CLI; a missing `gh` alone does not rule out another supported publishing route. Do not report PR delivery as complete without a PR URL.

Lead the report with the PR link and whether it is ready or draft, or the reason no PR was delivered. Include:

- what was built, against each done-when condition;
- the team that ran;
- advisor findings: fixed, and rejected with reasons;
- verification evidence, and anything outstanding;
- the integration branch.

Remove only worker worktrees this run created once their work is integrated and clean. Preserve the active integration checkout for the user's review. Release only locks this run added when their checkout no longer needs protection; retained active or blocked worktrees stay protected and are listed in the report.

## Wrapped skills

`/orchestrate /<skill> <args>` runs the other skill's process with this team. Steps 1 to 5 still apply. The wrapped skill decides **what** happens and in which order, replacing steps 6 to 11; this skill decides **who** does it. Preserve step 7's ownership, shared-resource and final-gate plan and step 8's conditional plan review before implementation, even when the wrapped workflow omits them. Consolidate an equivalent wrapped review rather than adding a duplicate. Apply step 12's single final gate and handoff after the wrapped workflow finishes. Whenever the wrapped skill (or a skill it loads) calls for a subagent, choose the role by purpose:

| The workflow asks for                             | Role       |
| ------------------------------------------------- | ---------- |
| implementing, fixing, a merge-conflict helper     | Worker     |
| a review, second opinion, standards or spec check | Advisor    |
| finding code, locating examples or prior art      | Explorer   |
| outside facts, docs, comparisons                  | Researcher |
| planning, integrating, reporting                  | You        |

Keep the wrapped skill's own rules: branching, tests, commits, review protocol and anything it requires in the report. When it keeps its own ledger, that is the run's only ledger: write its path in `ledger.md` and keep the run's state only in the wrapped skill's ledger. Apply [delegation.md](delegation.md) to every dispatch. When one of its instructions conflicts with running as a team (for example, "start in a fresh chat"), tell the user and follow their call.

Adopt and report any PR the wrapped skill already created instead of creating a duplicate; link it to T3 and record it in the run's actual ledger. When the wrapped workflow produces no repository changes, report its result without opening an empty PR. If its publishing rules conflict with the proposed delivery, settle that conflict with the user at confirmation, before dispatching work; honor an existing user decision rather than asking again.
