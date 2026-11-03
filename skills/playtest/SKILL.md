---
name: playtest
description: Playtest a game or app with a swarm of agent testers, each playing a different role, verify their findings with independent verifiers that try to refute them, and publish a report (optionally as issues). Also rechecks a prior playtest's findings against a new build, and sets up a repository for playtesting.
disable-model-invocation: true
---

# Playtest

Cheap testers use the product the way real users would, each from a different role, and propose findings. Strong verifiers try to refute each one. Only what survives reaches the user.

You coordinate: read the profile, plan the coverage, brief and launch agents, merge findings, run the verifiers, write the report. You are the only writer of the run's shared files.

## Words

- **Product**: the game or app under test.
- **Harness**: how an agent drives the product without a person: a headless game runner, a headless browser against a local stack, a CLI.
- **Session**: one complete use, from start to its end: a playthrough to an ending, or a journey to its goal (a Quote sent, an invoice paid).
- **Area**: a part of the product that coverage is planned in: an ending, the onboarding, one entity's lifecycle.
- **Reference session**: a known session, hidden from testers, that proves something can or can't be done. Verifiers use them to refute "impossible" claims.
- **Canon**: the documents that say how the product should behave: the design document, product rules, glossary, design system.

## The profile

Every repository describes itself in `.playtest/profile.md`, next to `.playtest/reference/` for its reference sessions. The profile starts from a template next to this skill (`templates/game.md` or `templates/app.md`) and holds everything about this product:

| Section | Says |
|---|---|
| Kind | `game` or `app`: which template's defaults apply |
| Product | what it is, who uses it, what a session's goal is |
| Harness | what you run once before testers start (build, local stack, dev server), the commands a tester uses, the loop of one session, and where its result is read |
| Isolation | how each tester gets its own state (its own account, fixture or save), and what testers must never touch |
| Serial steps | commands that may only run one at a time (GPU renders, database resets) |
| Hidden until first session | paths testers don't read before their first session is over |
| Canon | the documents to check against, and which values or rules count as open design decisions |
| Areas | the coverage areas |
| Roles | changes to the template's roles: drop, add, variants, extra personas (a game's opponent, an app's customer) |
| Presentation | how to capture the screen, and at which sizes |
| Evidence | what a finding cites and what a repro looks like, if not the template's |
| Reference sessions | what each file in `.playtest/reference/` proves, and how to run it |
| Issues | how findings become issues: milestone, labels, board |
| Models | tier changes for this repository, if any |

The template gives the defaults; the profile overrides them. Read both before planning a run. When the profile and this skill disagree about the product, the profile wins. When they disagree about the evidence bar or verification, this skill wins.

## Modes

Ask one question if the request doesn't make the mode clear.

- **Setup**: there is no `.playtest/profile.md` yet, or the user asks for one. See [Setup mode](#setup-mode).
- **Full** (default): testers, then verification, then a report.
- **Focused**: the same, limited to one area ("the endings", "the Quote flow", a PR's change). Pick the roles that fit the area. Coverage outside the focus is out of scope, never "fine".
- **Recheck**: rerun a prior run's confirmed findings against the current build. Each becomes fixed, still present or changed. No testers.

A full, focused or recheck run without a profile stops and offers setup.

## Sizes

| Size | Testers | Waves |
|---|---|---|
| quick | one per role | 1 |
| standard (default) | two per role, each second with a variant | 1, then a gap check |
| swarm | standard, plus a second wave of at most as many again | wave 2 goes where the gap check points; stop early if it finds little |

The presentation tester comes on top, in every size. Sessions per tester come from the template or the profile. Record the size in `run.json`. Sizes change breadth, never the evidence bar.

## Models

Models come in tiers: read [tiers.md](tiers.md) (or `~/.agents/tiers.md` when it exists) for the models and the rules. Every job here names a tier, not a model. A profile can move a job to another tier, or leave a model out for its repository.

| Job | Tier | How many models |
|---|---|---|
| Testers | swarm | all models that aren't reserves, round-robin. When there are two or more, the two testers of one role get different models, so one model's blind spot doesn't decide a role |
| Presentation tester | review | the first model |
| Gap check | mid | the first model |
| Verifiers | review | all of them, round-robin |
| Second check | review | a model from a different provider than the group's first verifier |

Record each agent's model in `run.json`.

### Harness probe

Before the first wave, each swarm model the run will use (reserves only when they are needed) plays one short session in the harness, with a probe brief: reach the first decision point, show the evidence the profile asks for, and stop. A model that can't drive the harness or return valid JSON is left out of this run and noted in `run.json` and the report. A reserve takes over only when the models above it fail the probe. If no swarm model passes, testers move up to the mid tier and the budget shrinks to match; say so before going on.

### Running them

Call `orchestrator_capabilities` first. Some harnesses attach the T3 tools lazily: when they are missing from your tool list, make one direct call by name before concluding. Without them, tell the user that `/playtest` needs T3 Code orchestration and stop.

Testers run one wave at a time. The presentation tester runs with the first wave. Verifiers run all at once.

## Budget

Count agent invocations per tier. Before launching any tester, reserve:
- review tier: the presentation tester, one verifier per expected finding group (about 2 per 3 testers, at least 6), and second checks for about 1 group in 4;
- mid tier: the gap check;
- swarm tier: the harness probe.

Spend the rest on testers. If verification runs out of budget, stop: unverified groups stay out of the report and are listed as unverified.

## Before testers

1. Run directory: `.scratch/playtest/run-<N>/`, the next unused N. Add `.scratch/` to `.gitignore` if it isn't there. The directory holds `run.json`, `coverage.json`, `findings.json`, `REPORT.md` and `testers/<id>/`.
2. Do the profile's one-time harness steps (build, local stack, dev server, fixtures). Testers never build, test, start servers or reset shared state.
3. Give each tester its own isolated state the way the profile's Isolation section says, before it starts. Note each tester's state (account, fixture, save) in `run.json`.
4. If a prior run exists, read its `findings.json`: the rejected fingerprints, and the confirmed ones for recheck.
5. Run the reference sessions. If one no longer holds on this build, note it in `run.json`; that is itself a finding for the report.
6. Write `coverage.json`, the plan of what must be covered. Units are role × area, from the profile's areas. Every unit gets an owner. A unit no tester gets is marked deferred, never dropped.

## Tester brief

Give each tester this word for word, plus its role, variant, units, session cap, its isolated state, and the profile's Harness, Isolation and Evidence sections pasted in:

- **Goal.** Use the product like a real user in your role and report what feels off.
- **Tool.** The harness, as pasted. Run each session to its end before judging it.
- **Blind first.** Until your first session is over, don't read the paths in the profile's "Hidden until first session" list. After that, read anything.
- **Your folder.** Write only in `testers/<id>/`. Use only your own state. Never build, test, start servers, reset shared state, commit, file issues, or touch what the Isolation section forbids. Run serial steps only when the coordinator says so.
- **The gate for "impossible" claims.** Before claiming something can't be done:
  - try at least two genuinely different ways and name them;
  - show the arithmetic or the exact screens and steps;
  - name the rule you think enforces the limit.
  If you can't, report "I couldn't find a way": a clarity finding, not an impossibility.
- **Evidence gate.** Each finding cites its evidence the way the profile's Evidence section says, and a repro that reproduces it.
- **Answer.** Exactly one JSON object as your final message, with no prose around it:

```json
{"tester": "<id>", "role": "<role>",
 "sessions": [{"repro": "testers/<id>/session-01.<ext>", "outcome": "<how it ended>", "notes": "…"}],
 "findings": [{"fingerprint": "<area>/<root-cause-slug>", "title": "…",
   "category": "bug|balance|pacing|flow|clarity|copy|accessibility|design-gap|tool", "proposed_severity": "high|medium|low",
   "claim": "…", "evidence": ["<file and place, with the numbers or what the screen showed>"],
   "repro": {"file": "testers/<id>/…", "command": "…"},
   "impossibility": false}],
 "worked_well": ["…"]}
```

The fingerprint is a short area/root-cause slug. It never includes a tester, date or severity. Use the area names from the profile.

The presentation tester gets the same brief, plus:
- capture 2–3 contrasting sessions the way the profile's Presentation section says, one at a time when that is a serial step;
- read every capture next to the moment it shows, at every screen size the profile lists;
- check wording and terms against the canon and the glossary.

Its fingerprints start with `presentation/`. If its harness refuses to write files, it returns everything as text.

## Merge

When a wave has reported:

1. Group findings by fingerprint, then by root cause: one root cause seen through several symptoms is one group. Note who raised each.
2. Drop groups that match a prior run's rejected fingerprint, unless the evidence is new.
3. Collect the session outcomes into `run.json`. When the harness has a summary command, run it over every tester's sessions.
4. Gap check (standard and swarm): one fresh tester gets `coverage.json`, the groups and the outcomes, and returns the units and areas nobody really exercised. In a swarm, aim wave 2 there.

How many testers raised a finding is not evidence. Five can share a misreading, and the finding only one raised can be the important one.

## Verify

One fresh review-tier verifier per group, all in parallel. A verifier never tested, and never sees another verifier's verdict. Give it the group, the profile, and its own isolated state when the harness needs one. Its prompt:

> You did not find this. Try to refute it.
> 1. Reproduce it with its repro. Rerun with the variations that would make it go away.
> 2. Read the code and the canon sections that decide it.
> 3. For an impossibility claim, run the reference sessions in `.playtest/reference/`. Testers never see them.
> 4. Decide:
>    - **confirmed**: real, with the evidence that proves it. Give a severity (anchors below) and the smallest fix.
>    - **needs-decision**: real, but fixing it changes an open design value, the canon or the design. Give what you measured and two or three options.
>    - **rejected**: refuted by a rerun, the code or the canon. Say exactly how.
>
> Return exactly one JSON object, with no prose around it:
> `{"fingerprint": "…", "decision": "confirmed|needs-decision|rejected", "severity": "high|medium|low|null", "record": {"title": "…", "what": "…", "evidence": "…", "repro": "…", "fix_or_options": "…"}, "reason": "…"}`

Every high finding, and every verdict that changes a group's root cause, gets a second check from a review-tier model of another provider, with the same prompt and without the first verdict. If the two disagree, report it as needs-decision with both readings.

Discard a prose-wrapped answer and rerun with a fresh verifier if the budget allows.

### Severity anchors

- **high**: a typical user can't reach their goal, or can't tell why they failed; data is lost or corrupted; another user's data is exposed; an exploit beats the product's rules; a rule in the canon is broken.
- **medium**: a feature that doesn't matter (its decision has no effect); a screen that misleads; dead time or needless friction in a common path.
- **low**: wording, polish, a minor inconsistency.

The template gives examples of each for its kind.

## Report

Write `findings.json` with every group's final record, rejected ones included, so the next run doesn't repeat them. Write `REPORT.md` and publish it as an artifact. It contains:

- who tested, in which roles, and the session outcomes;
- the top five;
- the confirmed findings by theme;
- the needs-decision findings as questions;
- what was rejected, and why;
- what coverage was deferred, and any reference session that no longer holds;
- a model scorecard: per tester model, the probe result, sessions finished, findings raised, and how many were confirmed, needs-decision and rejected. Per review model, the second checks where it disagreed with the first verifier.

Keep the scorecard in `run.json` too. Over several runs it shows which swarm models are worth their place in the swarm tier.

Use the product's names for things, never internal ids. Give the user the link, the counts and the decisions they need to make.

## Issues

Only when the user asks. Follow the profile's Issues section, and otherwise the repository's issue format (`AGENTS.md`, the contributing guide). By default: a milestone, a parent spec and sub-issues.
- Confirmed findings become issues ready for an agent.
- Needs-decision findings become questions.
- The parent spec names a reference session, so every issue can be reproduced.

## Recheck

For each confirmed fingerprint of the prior run, rerun its repro on the current build, and run the reference sessions. Each finding becomes:
- **fixed**: the evidence moved past the fix's bar;
- **still present**;
- **changed**: say how it behaves now.

Report it as a table. A regression in the session outcomes or the reference sessions is a finding too.

## Setup mode

Write `.playtest/profile.md` for this repository, for the user to confirm.

1. Read `README.md`, `AGENTS.md`, `CLAUDE.md`, the design docs and the test docs. Decide the kind and copy its template's profile skeleton.
2. Find the harness: a headless runner, a CLI, browser test tooling with local fixtures. If an agent can't drive the product without a person, say so, describe the smallest harness that would do, and stop. A playtest without a harness is guesswork.
3. Fill each section from what you read. Where the docs don't say (which paths stay hidden, which values are open decisions, what testers must never touch), ask; don't guess.
4. Propose two or three reference sessions: at least one that reaches the main goal, and one that fails in a known way. Write them into `.playtest/reference/` and run them.
5. Show the profile and the reference sessions' results. Commit nothing unless the user asks.

## Anti-patterns

1. Reporting an impossibility a reference session refutes.
2. Counting votes instead of verifying.
3. Settling an open design value in a fix. That is always needs-decision.
4. Prose findings that can't be merged.
5. Showing a verifier the tester's or another verifier's verdict.
6. Writing the report before verification is done.
7. Testers sharing state, or touching anything the Isolation section forbids.
8. Product details in this file. They belong in the profile.
