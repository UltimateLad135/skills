---
name: implement
description: Orchestrate the implementation of one issue, or every child ticket of a parent spec or milestone, with one subagent per ticket merged into an integration branch; then open a PR and run the council review.
disable-model-invocation: true
---

# Implement

You are the orchestrator. Subagents write the code. You plan the waves, review and merge each child's branch, keep the integration branch green, open one PR and run the council on it.

## Repository rules

Read `AGENTS.md`, `CLAUDE.md`, the contributing guide and the CI docs they point to. Settle:

- **The gate**: the commands that make up the complete check, in order. Include the check that regenerating generated files leaves no diff, when the repository has generators. When no doc names a gate, derive it from the CI workflow (`.github/workflows/`) and show it to the user once. Note whether a pre-push hook runs it.
- **Focused checks**: the fast checks for one change: typecheck, lint, and the tests of the areas it touches, using the per-area or selective commands the docs name. Children and waves run these. The complete gate runs once, on the finished integration head.
- **Hard rules**: constraints the docs say every change must keep, such as determinism, layering, security or data-integrity rules, how generated files are produced, required metadata such as asset provenance, and naming from a glossary. Quote them; don't summarize.
- **Readiness**: the label or field that marks an issue as fully specified for an agent (for example `ready-for-agent`), if the repository uses one.
- **Design docs**: where the specs live (a design document, ADRs, a spec folder), so you can point children at the right sections.
- **Workflow duties**: what the docs require of every change or PR beyond the code, such as moving a board card when work starts, adding a release note, sections a PR description must carry, or evidence a branch push leaves for a later gate. Quote them.

## Scope

The user names one of the following. When they name nothing, list the open issues that carry the readiness label, have no open blocker and aren't parent specs with open sub-issues, oldest first, one line each, and wait for a pick.

- **An issue.** That issue is the batch.
- **A parent spec.** The batch is its open sub-issues: `gh api repos/{owner}/{repo}/issues/<n>/sub_issues`. If they span several milestones, ask which milestone.
- **A milestone.** The batch is its open issues: `gh issue list --milestone "<title>" --state open`.

Read each issue with `gh issue view <n> --comments`, and the spec under its `## Parent` (or however the repository links a parent). Then check:

- When the repository uses a readiness label, an issue without it is not fully specified. Ask before taking it on.
- An issue blocked by one that is neither closed nor in the batch: stop and ask.
- Read only the design-doc sections the issues need.

Post the plan (waves, one line per issue with its child's model) and carry on without waiting. Do the workflow duties that belong to starting work, such as moving each issue's board card.

## Integration branch

Work on the current branch unless it is the default branch. In that case, create `implement/<slug>` first. Children branch from its head. Nothing is pushed until the PR.

## Waves

Order the batch by its blockers (`## Blocked by`, or the repository's dependency links). Wave 1 is every issue with no blocker left in the batch, wave 2 is what wave 1 unblocks, and so on. Issues in one wave run in parallel. A wave starts only after the previous wave is merged and its focused checks are green.

## Child models

Models come in tiers: read [tiers.md](tiers.md) (or `~/.agents/tiers.md` when it exists) for the models and the rules. Pick the best-value model for each issue yourself, without asking: the cheapest one you expect to get it right the first time. Any model in the tiers file is allowed, and its reserve rules apply. As a guide:

- **swarm** (DeepSeek V4.1 Flash first, then Gemini 3.8 Flash): small, fully specified issues in one area, such as copy, config, a simple component, or a test for existing behaviour.
- **mid** (for example Sonnet, Terra): the usual feature or bug fix.
- **review** (for example Opus, Sol): changes across several layers, concurrency, security or permissions, data migrations, money, or a subtle bug nobody has pinned down. Never give these a swarm-tier model.

With `boost` in the request, every child runs on the review tier, under the tiers file's Boost rules; still name each pick.

Spread picks across providers when they are equal in value, so one model's habits don't run through the whole batch. Name each pick and one line of why in the plan and in the PR description. When a child fails the review or the gate in a way a stronger model would have avoided, rerun the issue once on a model from the next tier up, and note it in the PR.

Without T3 Code, only your own provider's models are available: pick from those with your host's subagent tool (Claude Code's `Agent` with its `model` parameter).

## Child brief

One subagent per issue, each in its own worktree branched from the integration head: your host's subagent tool with worktree isolation (Claude Code's `Agent` with `isolation: "worktree"`, or T3 Code's `delegate_task` with `isolation: "worktree"`). The brief is all the context the child has, so it contains:

- the issue number and full body, plus the part of the parent spec it implements;
- the design-doc sections that matter, by heading, and the files it will likely touch;
- the rules: behaviour changes need tests; the repository's hard rules, quoted; generated files change only through their generators and the output is committed;
- the workflow duties that change files, such as a release note, quoted;
- stay inside the issue. Anything else it notices goes in its report, not its diff;
- if the issue needs a value or rule the spec leaves open (marked *tuning*, *TBD* or open), and the issue doesn't settle it, stop and report the question instead of picking one;
- run the focused checks, commit on its branch, and don't push or open a PR;
- report back: what changed, each acceptance criterion and how it is met (test name or manual check), deviations, open questions.

## Review and merge

For each finished child, before starting anything that depends on it:

1. **Review** the diff against the issue's acceptance criteria and the repository's hard rules. Read the code, don't trust the report. Check that the tests would fail without the change.
2. **Merge** with `git merge --no-ff`.
3. **Fix** what the review found yourself, in a separate commit whose message names the finding. Don't send it back to the child.
4. **Run the focused checks** for what the merge and fixes touched, on the integration branch.

Escalate to the user only when a finding, or a child's open question, touches an unsettled design question.

## Finish

You are done when every acceptance criterion of every issue in the batch is met and the focused checks are green on the integration head. Then:

1. Run the complete gate once on the integration head. When a pre-push hook runs it, the push is that run: don't run it by hand first. Fix what fails, and only then go on.
2. Push the branch and open a PR to the default branch with the description below. Put any section the repository requires in a PR description at the top, and add the evidence its workflow duties ask for.
3. Run the council: read `../council/SKILL.md` (next to this skill's folder) and follow it for this PR. When the council skill isn't installed, say so and stop after the PR.

```md
Implements <parent spec or milestone link>.

## Issues

- Closes #<n>: <what changed, one line> (<model>: <why that model>)

## Review fixes

- <finding> (#<issue>): <fix, one line>

## Deviations and open questions

- <anything that differs from an issue, or a question left for the user>
```

`Closes` only the batch's issues, never the parent spec. If merging would close the parent's last open sub-issue, tell the user: a project board may then move the parent to Done on its own.
