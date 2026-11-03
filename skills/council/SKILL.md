---
name: council
description: Review a pull request with a council of models from different providers, verify their findings, fix what is real and in scope, and post the verdicts on the PR.
disable-model-invocation: true
---

# Council

Several models review the same PR independently. You are the judge: you check every finding against the code, fix the real ones that are in scope, push back on the rest, and post the verdicts on the PR.

## Reviewers

Models come in tiers: read [tiers.md](tiers.md) (or `~/.agents/tiers.md` when it exists) for the models and the rules. The council is:

- every model in the **review** tier;
- the first model in the **swarm** tier, for a cheap outside view.

The more providers the council spans, the less its reviewers share blind spots.

Call `orchestrator_capabilities` first. Some harnesses attach the T3 tools lazily: when they are missing from your tool list, make one direct call by name before concluding. Without them, tell the user that `/council` needs T3 Code orchestration and stop. Run every reviewer that is available, each through `delegate_task`, all at once. If one is missing, note which and why, and go on with the rest.

## Before the reviewers

The user (or the implement skill) names a PR. Collect:

- `gh pr view <n>`: head branch, head SHA, description;
- the issues it closes, with their acceptance criteria;
- the repository's hard rules from `AGENTS.md`, `CLAUDE.md` and the docs they point to (determinism, layering, security, data integrity, generated files, required metadata), quoted, and its gate: the commands that make up the complete check.

Don't write down your own view of the PR before the reviewers report. It would leak into how you judge theirs.

## Reviewer brief

Every reviewer gets the same brief, word for word:

- the PR number and head SHA, and the closed issues' acceptance criteria, pasted in;
- how to read the change: `gh pr diff <n>`, and `git show <sha>:<path>` for whole files at the head (`git fetch origin` first if the SHA is missing). Read only: don't edit files, switch branches, commit, push or comment on the PR;
- what to look for, in this order:
  1. bugs and regressions in the changed code;
  2. breaches of the repository's hard rules, pasted in;
  3. when the diff crosses a trust boundary (a server endpoint or action, authentication or authorization, database access rules, money, stored files, secrets): what a lower-trust actor, such as another user or an anonymous visitor, can read or change that they shouldn't. Name the actor and the request;
  4. acceptance criteria that aren't met, or are "met" by tests that wouldn't fail without the change;
  5. layer violations the docs forbid (for example presentation code mutating domain state), generated files edited by hand, missing required metadata;
- what not to report: style, naming taste, refactors outside the diff, new features. A design concern gets one line under "Out of scope" at the end;
- the answer format, one block per finding. No finding without a concrete failure; "No findings" is a valid answer:

```md
### <short title>
- severity: blocker | major | minor
- location: <path>:<line>
- failure: <concrete input or state> → <wrong result>
- fix: <one line>
```

Wait for each reviewer through the `next` that `delegate_task` returns. Don't poll.

## Triage

Once every reviewer has reported or failed:

1. **Merge duplicates** and note which reviewers raised each finding.
2. **Verify each finding yourself** by reading the code, and run it when you can. The number of reviewers is not evidence: five can share a wrong idea, and the finding only one raised can be the important one.
3. **Give each a verdict:**
   - **Fixed:** real and in scope.
   - **Rejected:** wrong, or not a problem in this codebase. Say why.
   - **Deferred:** real but out of scope. List it for the user, don't fix it.

In scope means bugs, regressions, hard-rule breaches and unmet acceptance criteria in the code this PR changes. Everything else is scope creep, however good the idea. Anything that would change the project's design canon or settle a value the spec leaves open is always Deferred.

## Fix

Commit the fixes on the PR branch, one commit per finding or tight group, each message naming the finding. Run the gate, then push. When a pre-push hook runs the gate, the push is that run: don't run it by hand first.

One round only: don't run the council again on its own fixes. If a fix is big enough to deserve a second look, tell the user instead.

## Post

Post one comment on the PR (`gh pr comment <n> --body-file <file>`), then give the user the counts and the Deferred items in chat.

```md
## Council review

Reviewers: <each reviewer that ran, with its option level>.
Missing: <reviewer> (<reason>).

| # | Finding | Raised by | Verdict | Commit or reason |
|---|---------|-----------|---------|------------------|
| 1 | <one line> (`path:line`) | Grok, Sol | Fixed | <sha> |
| 2 | <one line> | DeepSeek | Deferred | <why; suggested follow-up> |
| 3 | <one line> | Gemini | Rejected | <why it isn't a problem> |

Gate after fixes: <green, or what failed>.
```

Order the rows Fixed, Deferred, Rejected, by severity within each. Leave out the "Missing" line when every reviewer ran.
