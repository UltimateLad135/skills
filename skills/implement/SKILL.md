---
name: implement
description: Orchestrate the implementation of one issue, or every child ticket of a parent spec or milestone, with one subagent per ticket merged into an integration branch; then open a PR and run the council review.
disable-model-invocation: true
---

# Implement

You are the orchestrator. Subagents write the code. You plan the waves, review and merge each child's branch, keep the integration branch green, open one PR and run the council on it.

## Repository rules

Read `AGENTS.md`, `CLAUDE.md`, the contributing guide and the CI docs they point to. Settle:

- **The gate**: the commands that make up the complete check, in order. Include the check that regenerating generated files leaves no diff, when the repository has generators. When no doc names a gate, derive it from the CI workflow (`.github/workflows/`) and show it to the user once.
- **Hard rules**: constraints the docs say every change must keep, such as determinism, layering, security or data-integrity rules, how generated files are produced, required metadata such as asset provenance, and naming from a glossary. Quote them; don't summarize.
- **Readiness**: the label or field that marks an issue as fully specified for an agent (for example `ready-for-agent`), if the repository uses one.
- **Design docs**: where the specs live (a design document, ADRs, a spec folder), so you can point children at the right sections.

## Scope

The user names one of:

- **An issue.** That issue is the batch.
- **A parent spec.** The batch is its open sub-issues: `gh api repos/{owner}/{repo}/issues/<n>/sub_issues`. If they span several milestones, ask which milestone.
- **A milestone.** The batch is its open issues: `gh issue list --milestone "<title>" --state open`.

Read each issue with `gh issue view <n> --comments`, and the spec under its `## Parent` (or however the repository links a parent). Then check:

- When the repository uses a readiness label, an issue without it is not fully specified. Ask before taking it on.
- An issue blocked by one that is neither closed nor in the batch: stop and ask.
- Read only the design-doc sections the issues need.

Post the plan (waves, one line per issue) and carry on without waiting.

## Integration branch

Work on the current branch unless it is the default branch. In that case, create `implement/<slug>` first. Children branch from its head. Nothing is pushed until the PR.

## Waves

Order the batch by its blockers (`## Blocked by`, or the repository's dependency links). Wave 1 is every issue with no blocker left in the batch, wave 2 is what wave 1 unblocks, and so on. Issues in one wave run in parallel. A wave starts only after the previous wave is merged and the gate is green.

## Child brief

One subagent per issue, each in its own worktree branched from the integration head: your host's subagent tool with worktree isolation (Claude Code's `Agent` with `isolation: "worktree"`, or T3 Code's `delegate_task` with `isolation: "worktree"`). When `/orchestrate` wraps this skill, its workers take this role. The brief is all the context the child has, so it contains:

- the issue number and full body, plus the part of the parent spec it implements;
- the design-doc sections that matter, by heading, and the files it will likely touch;
- the rules: behaviour changes need tests; the repository's hard rules, quoted; generated files change only through their generators and the output is committed;
- stay inside the issue. Anything else it notices goes in its report, not its diff;
- if the issue needs a value or rule the spec leaves open (marked *tuning*, *TBD* or open), and the issue doesn't settle it, stop and report the question instead of picking one;
- run the gate, commit on its branch, and don't push or open a PR;
- report back: what changed, each acceptance criterion and how it is met (test name or manual check), deviations, open questions.

## Review and merge

For each finished child, before starting anything that depends on it:

1. **Review** the diff against the issue's acceptance criteria and the repository's hard rules. Read the code, don't trust the report. Check that the tests would fail without the change.
2. **Merge** with `git merge --no-ff`.
3. **Fix** what the review found yourself, in a separate commit whose message names the finding. Don't send it back to the child.
4. **Run the gate** on the integration branch.

Escalate to the user only when a finding, or a child's open question, touches an unsettled design question.

## Finish

You are done when every acceptance criterion of every issue in the batch is met and the gate is green on the integration head. Then:

1. Push the branch and open a PR to the default branch with the description below.
2. Run the council: read `../council/SKILL.md` (next to this skill's folder) and follow it for this PR. When the council skill isn't installed, say so and stop after the PR.

```md
Implements <parent spec or milestone link>.

## Issues

- Closes #<n>: <what changed, one line>

## Review fixes

- <finding> (#<issue>): <fix, one line>

## Deviations and open questions

- <anything that differs from an issue, or a question left for the user>
```

`Closes` only the batch's issues, never the parent spec. If merging would close the parent's last open sub-issue, tell the user: a project board may then move the parent to Done on its own.
