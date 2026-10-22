---
name: land
description: Land the open PRs from parallel worktrees in one batch. Merges their heads onto one branch, resolves conflicts, runs the repository's full gate once on the combined tree and fast-forwards the base branch, so GitHub marks every PR merged. Defers any PR it cannot land with confidence.
disable-model-invocation: true
---

# Land

You are the **lander**. Several agents worked in parallel worktrees, each verified its own branch and opened a PR. You land them together: merge every ready PR onto one branch, verify the combined result **once**, then move the base branch to exactly the commit you verified.

Landing PRs one by one is what this skill replaces. Each host-side merge moves the base, so every later PR that conflicts or falls behind needs a fresh full gate, while PRs that merge cleanly leave a combined state on the base that nothing ever tested. A batch costs one gate, and the tree that lands is the tree that passed.

Invoking `/land` authorizes you to push the verified batch to the base branch and to update the PRs it lands. It does not authorize skipping hooks, force-pushing, rewriting PR branches, or landing a PR this skill says to defer.

```text
/land               every open PR that targets the default branch
/land 12 15 18      only these PRs
/land --plan        report the batch, order and expected conflicts; change nothing
```

## 1. Learn the repository's rules

Read `AGENTS.md`, `CLAUDE.md`, the contributing guide and the CI docs they point to. Settle, and write into the ledger (step 3):

- **Base branch and push remote**: the default branch unless the docs say otherwise.
- **The gate**: the command that is the complete check, and whether a mandatory `pre-push` hook runs it (`git config core.hooksPath`, `.husky/`, `.githooks/`). Note its expected duration, timeout guidance, shared resources such as a local database stack, and whether it **reuses a pass** for an already-verified tree, and on what key.
- **Fast checks**: a quick typecheck or lint you can run after each conflicted merge.
- **Merge preconditions**: required reviews, a preview deployment, hosted checks, labels.
- **Post-merge duties**: for example applying merged migrations to a hosted database. Run one only when the docs assign it to agents; otherwise list it in the report for the user.
- **Merge commit convention**: read `git log --merges -10` on the base branch and copy its message format.

Then check the host. Use `gh` when it is available. Read the base branch's protection and rulesets (`gh api repos/{owner}/{repo}/branches/<base>/protection` and `gh api repos/{owner}/{repo}/rules/branches/<base>`). A 404, an empty rule list, or the docs stating there is no protection means a direct push is allowed. A 403 can mean missing rights rather than no protection, so treat it as unknown unless its message says the plan does not offer protection. When protection forbids a direct push, or is unknown, deliver through one combined PR (step 7).

## 2. Choose the batch

List the open PRs targeting the base, or the ones named in the arguments, with `gh pr list --base <base> --state open --limit 100 --json number,title,headRefName,headRefOid,isDraft,reviewDecision,statusCheckRollup,labels,body,files`. A PR is **eligible** when all of these hold:

- it is not a draft, and its title and labels do not mark it as work in progress or "do not merge";
- no review requests changes, and every review the repository requires is given;
- every check the repository requires on its head has succeeded. Required means marked required by protection, or named by the docs, such as a preview deployment; other apps' checks reporting skipped or neutral do not count against it. Pending, missing or unknown is not success;
- its description lists no outstanding checks or unfinished work. When the docs say the pre-push hook runs the complete gate for PR branches, a pushed head counts as having passed it;
- it targets the base directly. A stacked PR is out of scope until its parent has landed and it has been retargeted.

Everything else is **deferred** with its reason. Deferring one PR never holds up the others.

Defer a PR whose ordered artifacts sort before ones already on the base, such as a migration dated before the base's newest migration. A local gate replays from scratch and passes it, but a hosted database that already applied the newer ones would refuse it or apply it out of order.

Order the eligible PRs so the batch stays easy to bisect and to resolve: PRs that others build on first, then ordered artifacts in their intended order (migrations by number or timestamp), then oldest first. Write down which PRs touch the same files; those are the likely conflicts.

Show the batch as a table: PR, title, order, eligible or deferred, reason, overlapping files. With `--plan`, stop here. Otherwise continue without asking, unless a decision below is needed.

**Fast path.** When the batch is a single PR whose head already contains the current base tip and passed the gate, its merged tree is already verified. Two kinds of head do not qualify and take the normal path: one that only passed a lighter tier of the gate, and one committed on the host by its update-branch button or web editor (committer `web-flow` on GitHub), which never ran a local hook. Otherwise merge it on the host with a merge commit pinned to the checked head (`gh pr merge <n> --merge --match-head-commit <sha>`), skip steps 3 to 7 and go to step 8.

## 3. Prepare the landing checkout

Create the run folder `<home>/.land/runs/<repo>/<run-id>/` with `ledger.md`. `<home>` is the absolute path of the user's home directory, because a literal `~` does not expand in every shell; the run id is the date plus a short slug. Record the base branch, the fetched base tip, the push remote, the delivery route and, per PR: number, head commit, order and state (`queued`, `merged-locally`, `deferred`, `landed`). Record every conflict resolution and every gate attempt there as you go.

Fetch the base, then create a dedicated worktree on a new branch from the fetched base tip:

```
git worktree add <home>/.land/runs/<repo>/<run-id>/checkout -b land/<run-id> <remote>/<base>
```

`land/<run-id>` is the **landing branch**; a rebuild (step 6) starts a fresh one, `land/<run-id>-2` and so on, from the same base tip in the same checkout, because a pushed landing branch is never rewritten. Use this one checkout for every merge, gate and push of the run. A gate's pass record usually lives in its checkout, so changing checkouts would lose it. Prepare the worktree the way the repository's docs describe (dependencies, ignored env files). Pass `-c rerere.enabled=true` to every `git merge` and `git commit` of the run, so a rebuilt batch replays your resolutions without changing the repository's config.

**Resume.** When `/land` finds an unfinished run for this repository, offer to resume it. Reconcile the ledger against the host before acting: PR states and heads, the base tip, and which landing branches were pushed. Where they disagree, the host wins.

## 4. Merge the batch

For each eligible PR in order:

1. `git fetch <remote> pull/<n>/head` and check the fetched commit equals the head recorded in step 2. A changed head goes back through step 2's checks.
2. `git merge --no-ff <head>` using the base's merge message convention, for example `Merge pull request #<n> from <owner>/<branch>` followed by the PR title. Merge the head commit itself, so the PR's own commits reach the base unchanged.
3. Resolve conflicts as below, then run the fast checks when the merge conflicted or touched files another PR in the batch changed.

**Resolve** conflicts that keep both sides' intent without judgment:

- imports, adjacent additions to the same list, registry, changelog or routing table, and formatting;
- lockfiles and generated files: take either side and regenerate them with the repository's own command, under any shared-resource lock its docs require. Never hand-merge them.

**Judge** conflicts where both sides changed the same logic. Resolve one only when both PRs' descriptions make each intent clear, the resolution keeps both, and the gate exercises the code. Otherwise `git merge --abort`, defer that PR with the conflicting files and the reason, and continue with the next one.

**Defer** without trying when a resolution would change an ordered artifact that a PR introduced, such as renumbering a migration, or touch security, payments, data deletion or other paths the repository's docs mark as sensitive.

List each resolved file and how you resolved it in the merge commit's body and in the ledger. When subagents are available (a host's subagent tool or T3 Code's `delegate_task`), send every judged resolution to one independent reviewer with the merge commit, both parents and both PR descriptions, and defer the PR if the reviewer finds the resolution loses either intent.

The step is done when every eligible PR is merged locally or deferred. When nothing merged, report and stop.

## 5. Verify once

Commit everything; the checkout must be clean. Run the complete gate once, on this exact commit, the way the repository runs it:

- **Pre-push hook that reuses a verified tree**: push the landing branch, `git push -u <remote> land/<run-id>`. The hook runs the full gate once, and hosted previews and checks build the combined tree. Wait for every check the repository requires before merging.
- **Pre-push hook without reuse**: go straight to step 7. The push to the base runs the gate once; there is no second push.
- **Local gate without a hook**: run the gate command here, then continue.
- **Hosted checks only**: push the landing branch and wait for its required checks.

Never pass `--no-verify` or bypass a hook unless the user says so. A full gate can take tens of minutes: run it in the background with the outer allowance the docs give, read new output while it runs, report elapsed progress, and leave the checkout untouched until it ends. A quiet wait on a shared-resource lock is neither a pass nor a failure.

## 6. When the gate fails

Read the failure before acting.

- **Infrastructure** (a stalled container, a lock timeout, a crashed runner): follow the docs' recovery and rerun once.
- **Attributable** to one PR: the failing test or code belongs to its changes, or to two PRs that only fail together. Defer that PR, or the later of the two, and rebuild the batch without it on a fresh landing branch from the recorded base tip, merging the remaining PRs again in order. `rerere` replays your resolutions.
- **Unclear**: bisect by gating the first half of the batch, then narrow down. Run the gate command directly in the landing checkout for these probes rather than pushing throwaway branches, which would start previews. Each step costs a full gate, so say how many you expect before starting.

Do not add fix commits of your own. The only edits you author are conflict resolutions inside merge commits, reviewed as in step 4; a failure that needs a code change defers the PR that needs it. Every rebuild is a new commit tree and needs a fresh gate. After two failed gates that did not narrow the cause, stop and report the failures, the deferred PRs and the ledger so the run can resume. Never land a batch whose current commit has not passed.

## 7. Land

1. Fetch the base and every PR head again. If a PR head moved since you merged it, the PR would not be marked merged and its new commits would be left out: rebuild without it and defer it. If the base moved, merge it into the landing branch, resolve as in step 4 and verify again.
2. Move the base to the verified commit with a fast-forward push: `git push <remote> HEAD:<base>`. With a pass-reusing hook this usually repeats the gate's key and finishes in seconds, but a missed reuse runs the full gate, so run it with the same background allowance as step 5. Never force-push. Read a rejection before acting: a declined hook goes to step 6, a non-fast-forward to 1, a protection refusal to 3.
3. When protection forbids a direct push, open one PR from the landing branch that lists every PR it lands and merge it with a **merge commit** once its required checks pass. Squash or rebase would drop the PR heads and leave the original PRs open.

## 8. Close out

1. Check each landed PR shows as **merged**. GitHub marks a PR merged once its head commit is in the base branch; allow it a minute. If one does not show as merged, report it; never close it as unmerged.
2. Check the issues its closing keywords (`Closes #n`, `Fixes #n`) name. Close any still open with a comment that links the PR, but only when the PR's description names that issue explicitly.
3. Delete the run's remote landing branches. Delete landed PR branches only when the repository's convention or its auto-delete setting does.
4. Run the post-merge duties the docs assign to agents, such as applying merged migrations, and report their result.
5. In T3 Code, link every landed and deferred PR to the thread with `link_pull_request`.
6. Remove the landing worktree. Leave the PRs' own worktrees alone; other threads may own them. List the ones now merged as safe for the user to remove.

Report, leading with what landed:

- landed PRs with the commit now on the base, and the base tip before and after;
- deferred PRs, each with its reason and what would unblock it;
- every conflict resolution, and which ones were judged and reviewed;
- the gate: the attempts, which stages ran, how long they took, and any bisection;
- issues closed, post-merge duties run, and duties left for the user;
- side effects the base push started, such as a production deployment.
