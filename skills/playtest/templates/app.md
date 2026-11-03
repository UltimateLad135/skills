# App template

Defaults for `kind: app`. The repository's `.playtest/profile.md` overrides any of them.

## Harness the app needs

- **A headless browser an agent drives from the shell**, such as `playwright-cli`: open a page, read an accessibility snapshot, click, type, take a screenshot, read the console and network log.
- **A local stack** the coordinator starts once: the dev server, and a local database and auth with disposable fixtures. Never production, a shared staging environment, or real email, payment or messaging providers. Outside hosts are blocked or stubbed, and outgoing mail is captured locally where testers can read it.
- **A fixture per tester**: a command that creates a fresh account (and tenant, workspace or studio) with the starting data the profile names, and prints its credentials. Testers never reset the database; one tester's data must not leak into another's session.

Testers share the coordinator's dev server. They never start their own.

## Sessions per tester

quick 2, standard and swarm 4. A session is one journey to its goal, and a tester's sessions should cover different journeys.

## Roles

| Role | Uses the app as | Looks for |
|---|---|---|
| newcomer | someone signing up today, with no guide | onboarding; empty states; whether the first task can be found; what the app assumes you know |
| daily user | someone doing the core work every day | the core journeys at speed; clicks and screens per task; whether lists, search and defaults fit real work |
| power user | someone with lots of data and edge cases | bulk actions, filters, long lists, unusual but valid data; the limits of each feature |
| careless | someone in a hurry who makes mistakes | wrong input, double submits, back button, refresh mid-flow, abandoned drafts, undo; whether errors say how to recover |
| breaker | a QA tester | every state of every entity, every transition; rules against the canon; the same data shown the same way on every screen; permissions |
| mobile | someone on a phone | small screens, touch targets, the installed PWA, flaky or no network |
| accessibility | someone using only the keyboard, or a screen reader | focus order, focus traps, labels, announcements, contrast, zoom to 200% |
| presentation (review tier) | a first-time user looking at the screen | layout, clutter, misleading numbers, missing feedback; copy and terms against the glossary and the design system |

Add personas the product has on another side of it, such as a customer who receives a quote, a guest who opens a shared link, or an admin. Their journeys usually start from something a tester's main persona sent.

Variants for second testers, for example:
- a starting point: an empty account; an account with a year of data;
- a habit: keyboard only; always on the slowest path; never reads help text;
- a focus: one journey end to end; one entity's whole lifecycle.

## Areas

Signup and sign-in, onboarding, each core journey, each entity's lifecycle (create, edit, every state, delete or archive), settings, notifications and emails, money and documents, sharing and permissions, mobile, accessibility. Name them for the app in the profile.

## Evidence

A finding cites the screenshot in its folder, the URL, the step it happened at, and what the screen showed against what it should have: `testers/t2/s03-step07.png, /quotes/new, step 7: total shows €0,00 after adding a line`. Bugs add console errors and failed network requests from the browser's log.

A repro is a step file, the exact commands from the fixture to the failure, and how to run it: `{"file": "testers/t2/session-03.steps.md", "command": "<fixture command>, then each step in order"}`. Write the steps as you go, so the repro is the session itself.

## Presentation

Screenshots of 2–3 contrasting journeys at each size, read next to the step they show. Default sizes: 390×844 (phone), 1280×800 and 1920×1080.

## Reference sessions

Step files in `.playtest/reference/`, each starting from a named fixture: one per core journey that reaches its goal, and one for each thing the product deliberately forbids. A reference that no longer runs is a finding.

## Severity examples

- **high**: a user can't finish a core journey, or thinks it finished when it didn't; data is lost, wrong or saved twice; one account sees another's data; money or legal documents come out wrong.
- **medium**: a screen that misleads; an error that doesn't say how to recover; a common task that takes far more steps than it should; a feature that is unreachable or does nothing.
- **low**: wording, alignment, a minor inconsistency.

Pricing, limits, legal wording and anything the canon leaves open are design decisions: changing them is always needs-decision.

## Profile skeleton

```md
# Playtest profile

Kind: app

## Product
<What the app is, who uses it, what their main goals are.>

## Harness
Once, before testers: `<start local stack>`, `<start dev server>` on `<url>`.
Testers use: `<browser CLI>`; see `<testing doc>`.
One session: create your fixture, sign in, do one journey to its goal, writing each step to your step file.

## Isolation
Per tester: `<fixture command>` prints a fresh account's credentials.
Never: production or hosted URLs, real providers (<email, payments>), resetting the database, other testers' accounts.

## Serial steps
<Commands that may only run one at a time, if any.>

## Hidden until first session
`<source folder>`, `<spec docs>`.

## Canon
`<product rules doc>`, `<design system doc>`, `<glossary>`. Open design values: <pricing, limits, …>.

## Areas
<signup>, <onboarding>, <each core journey>, <each entity>, <settings>, <emails>, mobile, accessibility.

## Roles
<Changes to the template's roles, extra personas, and the variants to use.>

## Presentation
Sizes: 390×844, 1280×800, 1920×1080. Language: <UI language>.

## Reference sessions
- `reference/<journey>.steps.md`: reaches <goal> from fixture <name>.
- `reference/<forbidden>.steps.md`: shows <thing> is refused.

## Issues
<Milestone, labels, board.>
```
