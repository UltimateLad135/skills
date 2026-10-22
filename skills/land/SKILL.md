---
name: land
description: Land the open PRs from parallel worktrees in one batch. Merges their heads onto one branch, resolves conflicts, runs the repository's full gate once on the combined tree and fast-forwards the base branch, so GitHub marks every PR merged. Defers any PR it cannot land with confidence.
disable-model-invocation: true
---

# Land

You are the **lander**. Several agents worked in parallel worktrees, each verified its own branch and opened a PR. You land them together: merge every ready PR onto one branch, verify the combined result **once**, then move the base branch to exactly the commit you verified.

Landing PRs one by one is what this skill replaces. Each host-side merge moves the base, so every later PR that conflicts or falls behind needs a fresh full gate, while PRs that merge cleanly leave a combined state on the base that nothing ever tested. A batch costs one gate, and the tree that lands is the tree that passed.

Invoking `/land` is the user's decision to merge the eligible PRs: it authorizes you to push the verified batch to the base branch and to update the PRs it lands. It does not authorize skipping hooks, force-pushing, rewriting PR branches, merging on the host, or landing a PR this skill says to defer.

```text
/land               every open PR that targets the default branch
/land 12 15 18      only these PRs
/land --plan        report the batch, order and expected conflicts; change nothing
```

## 1. Learn the repository's rules

Read `AGENTS.md`, `CLAUDE.md`, the contributing guide and the CI docs they point to. Settle, and write into the ledger (step 3):

- **Base branch and push remote**: the default branch unless the docs say otherwise.
- **The gate**: the command that is the complete check, and whether a mandatory `pre-push` hook runs it (`git config core.hooksPath`, `.husky/`, `.githooks/`). Note its expected duration, timeout guidance, shared resources such as a local database stack, whether it **reuses a pass** for an already-verified tree and on what key, and whether it runs a lighter tier for some refs.
- **Fast checks**: a quick typecheck or lint you can run after each conflicted merge.
- **Merge preconditions**: required reviews, a preview deployment, hosted checks, labels, and anything the docs ask the owner to check before merging, such as user-facing copy.
- **Known flaky tests**: the ones the docs, issues or recent gate logs name.
- **Deploys and post-merge duties**: what a push to the base starts (for example a production deployment), and duties such as applying merged migrations to a hosted database, with their preconditions and any dry-run mode. Run a duty only when the docs assign it to agents; otherwise list it in the report for the user.
- **Merge commit convention**: read `git log --merges -10` on the base branch and copy its message format.

Then check the host. Use `gh` when it is available. Read the base branch's protection and rulesets (`gh api repos/{owner}/{repo}/branches/<base>/protection` and `gh api repos/{owner}/{repo}/rules/branches/<base>`). A 404, an empty rule list, or the docs stating there is no protection means a direct push is allowed. A 403 can mean missing rights rather than no protection, so treat it as unknown unless its message says the plan does not offer protection. When protection forbids a direct push, or is unknown, the delivery route is one **combined PR** (steps 5 and 7); settle the route now.

## 2. Choose the batch

List the open PRs targeting the base, or the ones named in the arguments, with `gh pr list --base <base> --state open --limit 100 --json number,title,headRefName,headRefOid,isDraft,reviewDecision,statusCheckRollup,labels,body`. Fetch each head (`git fetch <remote> pull/<n>/head`) and read its changed files and commits from git (`git diff --name-only <remote>/<base>...<head>`, `git rev-list <remote>/<base>..<head>`) rather than from the host, which truncates large PRs. A PR is **eligible** when all of these hold:

- it is not a draft, and its title and labels do not mark it as work in progress or "do not merge";
- no review requests changes, and every review the repository requires is given;
- every check the repository requires on its head has succeeded. Required means marked required by protection, or named by the docs, such as a preview deployment; other apps' checks reporting skipped or neutral do not count against it. Pending, missing or unknown is not success;
- its description lists no outstanding checks or unfinished work;
- it targets the base directly. A stacked PR is out of scope until its parent has landed and it has been retargeted;
- its commits do not include another open PR's commits unless that PR is also eligible, in which case it lands first. Landing a PR that contains a deferred PR's work would land that work too;
- its ordered artifacts sort after the ones already on the base, such as a migration dated after the base's newest migration. A local gate replays from scratch and would pass an older one, but a hosted database that already applied the newer ones would refuse it or apply it out of order.

Everything else is **deferred** with its reason. Deferring one PR never holds up the others.

Order the eligible PRs so the batch stays easy to bisect and to resolve: PRs that others build on first, then ordered artifacts in their intended order (migrations by number or timestamp), then oldest first. Write down which PRs touch the same files; those are the likely conflicts.

Show the batch as a table: PR, title, order, eligible or deferred, reason, overlapping files. Below it, quote every item an eligible PR asks the owner to check, such as a user-facing text, and ask the user once to confirm those PRs; land only the ones they confirm and defer the rest. Without such items, continue without asking. With `--plan`, stop after the table.

A batch of one PR takes the same route. When the repository's gate reuses a pass that is visible to the landing checkout, an up-to-date PR's merged tree can repeat its key and cost seconds; otherwise it costs one gate. Never merge on the host instead: the base can move between your check and the host's merge, which would land an untested tree.

## 3. Prepare the landing checkout

Create the run folder `<home>/.land/runs/<repo>/<run-id>/` with `ledger.md`. `<home>` is the absolute path of the user's home directory, because a literal `~` does not expand in every shell; the run id is the date plus a short slug. Record the base branch, the fetched base tip, the push remote, the delivery route, the current landing branch and, per PR: number, head commit, order and state (`queued`, `merged-locally`, `deferred`, `landed`). Record every conflict resolution and every gate attempt there as you go, and refresh the ledger's `updated` time at each step.

Fetch the base, then create a dedicated worktree on a new branch from the fetched base tip:

```
git worktree add <home>/.land/runs/<repo>/<run-id>/checkout -b land/<run-id> <remote>/<base>
```

The branch currently checked out there is the **landing branch**. A rebuild (step 6) starts a fresh one in the same checkout, `git switch -c land/<run-id>-<n> <base tip>`, because a pushed landing branch is never rewritten; record each in the ledger and use the current one in every command. Use this one checkout for every merge, gate and push of the run: a gate's pass record usually lives in its checkout, so changing checkouts would lose it. Before the first merge, prepare the worktree the way the repository's docs describe (dependencies, ignored env files and their copy helper). Pass `-c rerere.enabled=true` to every `git merge` and `git commit` of the run, so a rebuilt batch replays your resolutions without changing the repository's config.

**One run at a time.** Before starting, look for other runs of this repository. A run whose ledger is unfinished and was updated in the last hour may still be live: ask the user before touching it, and never start a second run beside it. Otherwise offer to resume the unfinished run. Reconcile its ledger against the host before acting: PR states and heads, the base tip, and which landing branches were pushed. Where they disagree, the host wins. When the verified landing commit is already on the base, only step 8 remains.

## 4. Merge the batch

For each eligible PR in order:

1. Check the fetched head still equals the head recorded in step 2. A changed head goes back through step 2's checks.
2. `git merge --no-ff <head>` using the base's merge message convention, for example `Merge pull request #<n> from <owner>/<branch>` followed by the PR title. Merge the head commit itself, so the PR's own commits reach the base unchanged.
3. Resolve conflicts as below and commit, then run the fast checks when the merge conflicted or touched files another PR in the batch changed.
4. When the resolution was judged, have it reviewed (below) **before** merging the next PR.

**Resolve** conflicts that keep both sides' intent without judgment:

- imports, adjacent additions to the same list, registry, changelog or routing table, and formatting. Check any invariant the docs give for such a list, such as unique ids or date order;
- lockfiles and generated files: resolve every other conflicted file first, then take either side and regenerate these with the repository's own command, under any shared-resource lock its docs require, as the last step before committing. Never hand-merge them, and regenerate again on every rebuild rather than trusting `rerere` to replay them.

**Judge** conflicts where both sides changed the same logic. Resolve one only when both PRs' descriptions make each intent clear, the resolution keeps both, and the gate exercises the code. Otherwise `git merge --abort`, defer that PR with the conflicting files and the reason, and continue with the next one.

**Defer** without trying when a resolution would change an ordered artifact that a PR introduced, such as renumbering a migration, or touch security, payments, data deletion or other paths the repository's docs mark as sensitive.

List each resolved file and how you resolved it in the merge commit's body and in the ledger. When subagents are available (a host's subagent tool or T3 Code's `delegate_task`), send every judged resolution to one independent reviewer with the merge commit, both parents and both PR descriptions. When the reviewer finds the resolution loses either intent, undo that merge on the unpushed landing branch (`git reset --hard HEAD~1`, then `git rerere forget <path>` for each judged file) and defer the PR. Without a reviewer, flag judged resolutions in the report.

The step is done when every eligible PR is merged locally or deferred. When nothing merged, report and stop.

## 5. Verify once

Commit everything; the checkout must be clean. Run the complete gate once, on this exact commit, the way the repository runs it:

- **Pre-push hook that reuses a verified tree**: push the landing branch, `git push -u <remote> <landing branch>`. The hook runs the full gate once, and hosted previews and checks build the combined tree.
- **Pre-push hook without reuse**: go straight to step 7. The push to the base runs the gate once; there is no second push.
- **Local gate without a hook**: run the gate command here, then continue.
- **Hosted checks only**: push the landing branch and wait for its required checks.
- **Combined PR route**: push the landing branch, open the combined PR (step 7.4) now so checks that run on pull requests start, and wait for its required checks.

Read the required checks from the **landing commit** itself (`gh api repos/{owner}/{repo}/commits/<sha>/status` and `.../check-runs`), never from the original PRs' rollups: those describe other trees. A preview the repository's ignore rule skips counts as the docs say. A failed required check is a failed gate (step 6).

A hook that runs a lighter tier for some refs must run its complete tier for this push; when the docs say a landing branch gets only the light tier, run the complete gate command here as well.

Never pass `--no-verify` or bypass a hook unless the user says so. A full gate can take tens of minutes: run it in the background with the outer allowance the docs give, read new output while it runs, report elapsed progress, and leave the checkout untouched until it ends. A quiet wait on a shared-resource lock is neither a pass nor a failure.

## 6. When the gate fails

Read the failure before acting.

- **Infrastructure** (a stalled container, a shared-resource lock timeout, a crashed runner): follow the docs' recovery and rerun. When another run holds the shared resource, wait for it to finish rather than racing it. These do not count as failed gates below.
- **Known flaky**: a test the docs or earlier logs name as flaky fails and the batch did not change it or the code it covers. Rerun once before blaming a PR, and copy every flaky-test warning the gate prints into the report.
- **Attributable** to one PR: the failing test or code belongs to its changes, or to two PRs that only fail together. Defer that PR, or the later of the two, and rebuild.
- **Unclear**: bisect. Check out the landing branch's commit after its first half of merges (`git switch --detach <sha>`), run the gate command directly, then narrow down, and switch back to the landing branch before anything else. Pushing throwaway branches would start previews. Each probe costs a full gate, so say how many you expect before starting.

To **rebuild**, start a fresh landing branch from the recorded base tip (the new tip, recorded, when the base moved), merge the remaining PRs again in order (step 4), and return to step 5 for the gate and every required hosted check. `rerere` replays the resolutions it recorded; regenerate generated files yourself.

Do not add fix commits of your own. The only edits you author are conflict resolutions inside merge commits, reviewed as in step 4; a failure that needs a code change defers the PR that needs it. After two failed gates that did not narrow the cause, stop and report the failures, the deferred PRs and the ledger so the run can resume. Never land a commit that has not passed.

## 7. Land

1. **Recheck.** Fetch the base and every PR head again, and repeat step 2's checks for every PR in the batch: draft state, reviews, labels and ordered artifacts against the current base. Then check that no open PR outside the batch has its head inside the landing commit (`git merge-base --is-ancestor <head> HEAD`); if one does, the host would mark it merged too, so defer the PR that contains it and rebuild. A PR whose head moved would not be marked merged, and its new commits would be left out. If a PR is no longer eligible or its head moved, defer it and rebuild. If the base moved, rebuild from the new base tip with every still-eligible PR; do not merge the base into the landing branch, which would put other work on the landing's first-parent line. Each rebuild returns to step 5.
2. **Preflight.** When the batch triggers post-merge duties the docs assign to agents, check their preconditions now, before the base moves: for hosted migrations, the backup age, the hosted history and any dry run the docs provide. If a precondition fails, rebuild without the PRs that need it, or stop and ask when the user must fix it first. A deployment from the base can go live before a migration runs, so new code must not wait on a duty that may fail.
3. **Push.** Check that the checkout is clean and `HEAD` is the tip of the landing branch that passed. Then move the base to it with a fast-forward push: `git push <remote> HEAD:<base>`. With a pass-reusing hook this usually repeats the gate's key and finishes in seconds, but a missed reuse runs the full gate, so run it with the same background allowance as step 5. Never force-push. Read a rejection before acting: a declined hook goes to step 6, a non-fast-forward to 1, a protection refusal to 4.
4. **Combined PR route.** When a direct push is not allowed, open one PR from the landing branch that lists every PR it lands, and merge it with a **merge commit**, pinned to the verified head (`gh pr merge <n> --merge --match-head-commit <sha>`), once its required checks pass. Squash or rebase would drop the PR heads and leave the original PRs open. Recheck as in 1 first; if the base moved, rebuild.

## 8. Close out

1. Run the post-merge duties right away, then check what the base push deployed, where the host or docs show it, and report its state.
2. Check each landed PR shows as **merged**. GitHub marks a PR merged once its head commit is in the base branch; allow it a minute. If one does not, its head moved after the recheck: its newer commits did not land, so leave it open and report it. Never close a PR as unmerged.
3. Let the host's own automation settle for a minute, then, for each PR that shows merged, check the issues its closing keywords (`Closes #n`, `Fixes #n`) name. Close any still open with a comment that links the PR, but only when the PR's description names that issue explicitly.
4. Delete the run's remote landing branches. Delete landed PR branches only when the repository's convention or its auto-delete setting does.
5. In T3 Code, link every landed and deferred PR to the thread with `link_pull_request`.
6. Remove the landing worktree and mark the ledger finished. Leave the PRs' own worktrees alone; other threads may own them. List the ones now merged as safe for the user to remove.

Report, leading with what landed:

- landed PRs with the commit now on the base, and the base tip before and after;
- deferred PRs, each with its reason and what would unblock it;
- every conflict resolution, and which ones were judged and reviewed;
- the gate: the attempts, which stages ran, how long they took, flaky-test warnings, and any bisection;
- issues closed, post-merge duties run, and duties left for the user;
- what the base push deployed, and its state.
