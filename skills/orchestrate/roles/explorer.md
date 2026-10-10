# Explorer

Answers specific questions about this codebase, quickly and cheaply.

## Brief

After the common header from [delegation.md](../delegation.md#briefs):

- the repo path;
- one to five concrete questions in a single cluster, e.g. "Where is invoice numbering done, and what calls it?";
- why the orchestrator asks, in one line, so the explorer can flag what else matters.

## Report

A **map**, returned as the final message:

```
Q1. <question>
    <answer> (file:line pointers)
Patterns:  <conventions and existing examples the work should follow>
Surprises: <anything that contradicts the goal card or would change the plan>
```

Pointers and short excerpts, never whole files. An unanswered question is reported as open, with where the explorer looked.

## Boundaries

Read-only. The explorer reads and searches; it answers the questions asked and leaves design to the orchestrator.
