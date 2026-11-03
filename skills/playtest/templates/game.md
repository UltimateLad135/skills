# Game template

Defaults for `kind: game`. The repository's `.playtest/profile.md` overrides any of them.

## Harness the game needs

A headless runner an agent can play through, without the engine's window:

- **describe**: the map, rules, starting state and briefing a player would see;
- **play**: run a plan of timed orders and stop at the game's own pauses (or at a named time), so the tester reads the situation and adds orders, the way a player would;
- **report**: what the player could see where the run stopped, how each order fared, and the numbers over time;
- **summary** (recommended): one line per plan played to the end, to compare many testers' runs.

The runner must be deterministic: the same plan, seed and options give the same run, so any session can be reproduced from its plan.

## Sessions per tester

quick 6, standard and swarm 10.

## Roles

| Role | Plays as | Looks for |
|---|---|---|
| sensible | a careful player trying to win well | whether good play is rewarded; dominant openings; the difficulty curve |
| passive | a casual player who acts only when the game stops, with the smallest fix | pacing; whether the game prompts; whether warnings come in time |
| optimizer | a min-maxer | exploits; dominant or useless options and purchases; farmable scores |
| newcomer | a beginner who makes the usual mistakes, then recovers | onboarding; refusal messages; whether cause and effect are visible |
| breaker | a QA tester | every ending reached on purpose; actions on edge ticks; conservation; rules against the design doc |
| micromanager | someone who pauses constantly and reshuffles before each big moment | whether attention is rewarded or required; how much input good play costs; fiddly loops |
| comeback | a player who falls behind on purpose, then recovers | whether a run is decided early; recovery rates; the difficulty curve |
| presentation (review tier) | a first-time player looking at the screen | clutter, legibility, misleading numbers, missing feedback; wording against the canon |

Drop micromanager in a turn-based game without pauses, and comeback in a short game that can't be lost early.

Variants for second testers, for example:
- an opening: one strategy first; one option only;
- a habit: spend everything; never use one system;
- a focus: one resource; the hardest moment.

## Areas

Opening, the first big challenge, mid-game, each ending, each economy or resource system, each major option or unit type, the HUD. Name them for the game in the profile.

## Evidence

A finding cites a report file and section with the numbers: `testers/t3/report-03.md: Routes, #4 7/0/0/0, 0 t/day`.

A repro is a plan file and the command that plays it: `{"file": "testers/t3/plan-02.json", "command": "<runner> testers/t3/plan-02.json --until=…"}`.

## Presentation

The runner, or the engine, renders a session to screenshots: replay 2–3 contrasting plans and capture the key moments. Rendering usually needs the GPU, so it is a serial step. Default sizes: 1600×900 and 1920×1080.

## Reference sessions

Plan files in `.playtest/reference/`: one that wins or holds with good play, and one that fails in a known way. Retune them when balance changes; a reference that no longer holds is a finding.

## Severity examples

- **high**: a typical player can't win, or can't tell why they lost; an exploit that beats intended throughput; a broken rule.
- **medium**: a system whose decisions have no effect; a screen that misleads; long stretches with nothing to do.
- **low**: wording, polish, a minor inconsistency.

Balance numbers that the design doc marks as tuning are open design values: changing them is always needs-decision.

## Profile skeleton

```md
# Playtest profile

Kind: game

## Product
<What the game is, who plays it, how a run ends.>

## Harness
Once, before testers: `<build command>`.
Testers use: `<describe command>`, `<play command> plan.json --pause`, `<summary command>`.
One session: describe, write opening orders, play to the next pause, read the report, add orders, repeat until the run is over.
See `<runner README>`.

## Isolation
Every run is independent; testers only need their own folder.
Never: <launch the editor, change game data, …>.

## Serial steps
`<screenshot command>` (uses the GPU).

## Hidden until first session
`<game data folder>`, `<design doc sections with the numbers>`, `<engine folder>`.

## Canon
`<design doc>`, `<glossary>`. Open design values: <numbers the design doc marks as tuning>.

## Areas
<opening>, <first challenge>, <mid-game>, <ending A>, <ending B>, <economy>, <each option>, HUD.

## Roles
<Changes to the template's roles, and the variants to use.>

## Presentation
`<render command>`, then `<screenshot command>`. Sizes: 1600×900, 1920×1080.

## Reference sessions
- `reference/<good-plan>.json`: holds with good play.
- `reference/<broken-plan>.json`: fails in a known way.
Run with `<summary command> .playtest/reference/*.json`.

## Issues
<Milestone, labels, board.>
```
