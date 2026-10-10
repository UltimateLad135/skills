# Advisor

An independent senior check at fixed checkpoints. The advisor advises; the orchestrator decides.

## When

- **Plan review**: before implementation when step 8 requires it, including wrapped workflows.
- **Code review**: on the integrated branch, before anything is pushed (step 11).
- **Follow-up check**: after accepted findings are fixed, on those changes only.
- **Consult**: when the orchestrator is stuck, such as a task that failed twice or workers whose reports conflict.

Each checkpoint gets one full round and one follow-up check. Further rounds need the user.

## Brief

After the common header from [delegation.md](../delegation.md#briefs):

- the checkpoint and what to judge;
- the goal card;
- the material: the plan, or the integrated diff against the base commit (`git diff <base>...<integration branch>`) plus focused verification evidence and the pending final gate;
- for a follow-up check: the earlier findings, how each was resolved, the fix diff, and unresolved objections.

Leave worker conversations and reasoning out of the brief. The advisor judges the result against the goal card, which keeps its view independent.

## Report

```
Verdict: go | go with changes | stop

1. [critical|major|minor] <title>
   Where:    <file:line, or plan task id>
   Evidence: <what shows the problem>
   Fix:      <concrete change>
```

Findings are ranked by severity, most severe first. Each one is backed by evidence the orchestrator can check. A `stop` verdict names what must change before the work can continue.

## Boundaries

Read-only. The advisor reads code and runs read-only commands, and returns findings and suggested fixes; workers make the changes.
