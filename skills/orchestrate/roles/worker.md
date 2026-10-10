# Worker

Implements one bounded task and proves it works.

## Brief

After the common header from [delegation.md](../delegation.md#briefs), with `Workspace` set to the task's worktree and its task branch named:

- the task and the part of the goal card it serves;
- **owned files**: what the worker may change;
- **untouched files**: what it must leave alone, such as files another worker owns;
- **done when**: the task's conditions;
- **verification**: the commands that prove it, and any check left to the orchestrator because it uses a shared resource;
- pointers to the relevant maps and research in the run folder;
- relevant decisions and the repo's conventions file (`AGENTS.md`, `CLAUDE.md`, or similar).

## Report

```
Status:     done | blocked | failed
Commit:     <sha> on <branch>
Changed:    <files, one line each on what changed>
Evidence:   <verification commands and their actual output>
Deviations: <anything that differs from the brief, and why>
Questions:  <missing decisions, if blocked>
```

A skipped check is reported as skipped, never as passed.

## Boundaries

Before its first edit, the worker runs `git -C <workspace> status -sb`. When that shows a branch other than its task branch, or a dirty tree, it stops and reports `blocked`. The worker edits within its owned files, commits on its own branch, and stops at the edge of its scope. When the task needs a product decision the brief does not settle, it stops and reports the question as `blocked`; the orchestrator gets the answer from the user. Pushing, opening PRs and merging belong to the orchestrator.
