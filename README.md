# Agent skills

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
2. **Batch**: eligible PRs (not draft, no requested changes, required checks green, nothing outstanding, no deferred work in their history, no migrations older than the base's) in a bisect-friendly order; everything else is deferred with a reason. Anything a PR asks the owner to check, such as user-facing copy, is quoted and confirmed with you once.
3. **Merge** each PR head with `--no-ff` in a dedicated worktree. Mechanical conflicts are resolved, lockfiles and generated files regenerated; a conflict that needs judgment is resolved only when both intents are clear, otherwise that PR is deferred and the rest continue.
4. **Gate once** on the combined commit. With a pre-push hook that reuses a verified tree, pushing the landing branch runs the gate and builds previews, and the push to the base then costs seconds.
5. **On failure**: rerun infrastructure and known-flaky failures, defer the PR that caused a real one and rebuild, or bisect when the cause is unclear. The lander never writes fixes of its own.
6. **Land**: recheck every PR and the base (a moved base means a rebuild and a fresh gate), check post-merge preconditions such as a hosted migration's backup, then fast-forward the base, or merge one combined merge-commit PR when protection forbids direct pushes. Never `--no-verify`, never force-push, never a host-side merge of an unverified combination.
7. **Close out**: run post-merge duties right away, check what deployed, confirm each PR shows merged, close issues it names, and report what landed, what was deferred and why, every conflict resolution and the gate evidence.

Each run keeps its ledger and landing worktree in `~/.land/runs/<repo>/<run-id>/`, so an interrupted run can resume.

## `/implement`

Implement one issue, or every child ticket of a parent spec or milestone, with one subagent per ticket, then open one PR and run `/council` on it.

### Requirements

Git, `gh`, and a host with subagents that can work in their own worktree (Claude Code's `Agent`, or T3 Code's `delegate_task`). The skill reads the gate and the hard rules every change must keep from the repository's `AGENTS.md`, `CLAUDE.md` and CI docs. Install `/council` too, or the run stops after the PR. The orchestrator picks the best-value model for each issue itself, and names each pick and why in the plan and the PR, from [Model tiers](#model-tiers): a cheap model for a small, fully specified issue, a mid-tier one for the usual feature, a review-tier one for security, migrations or work across layers.

### Install

```bash
npx skills add UltimateLad135/skills --skill implement
npx skills add UltimateLad135/skills --skill council
```

### Use

```text
/implement                       pick from the ready issues with no open blocker
/implement 42                    one issue, or a parent spec's open sub-issues
/implement milestone "Alpha 1"   every open issue in a milestone
```

### How a run goes

1. **Rules**: the gate and the focused checks, the hard rules, the readiness label, where the design docs live, and the workflow duties (board moves, release notes, required PR sections).
2. **Scope**: read each issue and its parent spec. Issues that aren't marked ready, or are blocked by something outside the batch, are raised with you first.
3. **Waves**: order the batch by its blockers. Each wave runs in parallel, one child per issue in its own worktree, branched from an `implement/<slug>` integration branch.
4. **Review and merge**: the orchestrator reads each child's diff against the acceptance criteria and the hard rules, merges it with `--no-ff`, fixes findings itself in separate commits and runs the focused checks.
5. **PR and council**: once every criterion is met, run the complete gate once (through the pre-push hook when there is one), push, open one PR that closes the batch's issues (never the parent spec), and run `/council` on it.

## `/council`

Review a PR with a council of models from different providers. The agent you are talking to judges every finding against the code, fixes what is real and in scope, and posts the verdicts on the PR.

### Requirements

T3 Code with orchestration (`delegate_task`) and at least one reviewer provider; outside T3 Code the skill says so and stops. The council is every model in the review tier plus the first swarm-tier model (see [Model tiers](#model-tiers)).

### Install

```bash
npx skills add UltimateLad135/skills --skill council
```

### Use

```text
/council 57    review PR #57
```

### How a run goes

1. **Brief**: every reviewer gets the same brief: the PR, the head SHA, the closed issues' acceptance criteria and the repository's hard rules. When the diff crosses a trust boundary (endpoints, auth, database access rules, money, stored files), they also look for what a lower-trust actor can read or change. They read only and report findings, each with a concrete failure.
2. **Triage**: merge duplicates, verify each finding by reading and running the code, and mark it Fixed, Rejected (with why) or Deferred (real but out of scope). How many reviewers raised a finding is not evidence.
3. **Fix**: one commit per finding on the PR branch, the gate once (the pre-push hook, when there is one), push. One round only.
4. **Post**: one PR comment with the verdict table, and the Deferred items in chat.

## `/playtest`

Playtest a game or an app with a swarm of agent testers, each in a different role, and get only the findings that survive independent verification.

Cheap testers use the product the way real users would: a careful player, a min-maxer and a newcomer in a game; a daily user, a careless user and a keyboard-only user in an app. Each reports findings with evidence and a repro. Fresh verifiers try to refute every finding, and a model from another provider checks the high ones.

The report holds what's confirmed, the design questions for you, and what was rejected and why. Testers run on the swarm tier, verifiers and the presentation tester on the review tier, the gap check on the mid tier (see [Model tiers](#model-tiers)). The report scores each tester model, so you can see which cheap models earn their place.

### Requirements

T3 Code with orchestration (`delegate_task`), and a harness through which an agent can use the product without a person: a headless game runner, or a headless browser such as `playwright-cli` against a local stack with a disposable account per tester. The models come from [Model tiers](#model-tiers); a repository's profile can change a job's tier.

Each repository describes itself in `.playtest/profile.md` (its harness, isolation, canon, areas and roles) with hidden reference sessions in `.playtest/reference/`. `/playtest setup` drafts both from the repository's docs.

### Install

```bash
npx skills add UltimateLad135/skills --skill playtest
```

### Use

```text
/playtest setup                  write the repository's profile and reference sessions
/playtest                        full run, standard size
/playtest quick the Quote flow   a focused run, one tester per role
/playtest swarm                  two waves, the second aimed at the gaps
/playtest recheck                rerun the last run's confirmed findings on this build
```

### How a run goes

1. **Profile**: read `.playtest/profile.md` and its template (game or app).
2. **Setup**: start the harness once, give every tester its own state, run the reference sessions, and plan coverage as role × area. Each swarm model plays one probe session first; a model that can't drive the harness sits the run out.
3. **Testers**: one wave in parallel, blind to the internals until their first session is over. Claims that something can't be done must name two attempts, the arithmetic and the rule.
4. **Merge**: group findings by root cause, drop ones a prior run already rejected, and check for coverage gaps; a swarm aims a second wave there.
5. **Verify**: one fresh verifier per group tries to refute it: confirmed, needs-decision or rejected. High findings get a second model's check.
6. **Report**: published as an artifact; issues only when you ask.

Runs live in `.scratch/playtest/run-<N>/`, gitignored, so the next run skips what was already rejected and a recheck knows what to rerun.

## Model tiers

`/playtest`, `/council` and `/implement` pick models by tier, from `tiers.md` next to each skill (identical copies, since each skill installs on its own):

| Tier | Models | Used for |
|---|---|---|
| **swarm**: cheap, run many | DeepSeek V4.1 Flash · max, Gemini 3.8 Flash · high; reserves: Claude Haiku 5.5 · max, GPT-6 Luna · max | playtest testers; council's outside view; implement children for small issues |
| **mid**: smarter, a few at a time | Claude Sonnet 5.5 · high, GPT-6.1 Terra · high | playtest gap check; implement children for the usual issue |
| **review**: strongest, spend carefully | Claude Opus 5.5 · high, GPT-6.1 Sol · xhigh, Gemini 4 Argon · high, Grok 4.7 · xhigh | council reviewers; playtest verifiers and presentation; implement children for hard issues |

To use other models everywhere, copy `tiers.md` to `~/.agents/tiers.md` and edit it: it replaces every skill's copy. A missing model is skipped and its jobs spread over the rest of its tier; a job never moves down a tier. Reserves run only when the models above them are missing or failing: Haiku and Luna share usage limits with the bigger Claude and GPT models.

## License

See [LICENSE](LICENSE).
