# Team selection

The user chooses a model and effort for every role. The orchestrator is this thread: its model is shown on the screens and never chosen here.

Ask with the host's structured question tool (such as `AskUserQuestion`); without one, ask in chat with the same content. The screens fit that tool's limits: up to four questions per screen, two to four options per question, an automatic free-text **Other**, and an optional preview for each option.

## Role needs

Suggestions come from the live catalog of `orchestrator_capabilities`, limited to providers with `canRunChildTask: true`. The catalog carries no quality or price data, so judge strength and cost from your knowledge of the model families, and label every cost as an estimate.

| Role | Needs | Default effort |
|---|---|---|
| Advisor | The deepest reasoning available, from a different family than the workers | high |
| Worker | Strong coding, scaled to difficulty: small → fast and cheap, medium → solid mid-tier, large → the strongest coder | medium |
| Explorer | Fast and cheap | low |
| Researcher | Web search and judgment about sources, mid-tier | medium |

## Screen 1: starting team

One question. Its text carries the difficulty card and lists the names of any saved teams. Options:

- **Last team: `<name>`** (recommended), when one is saved;
- **Balanced**, **Max quality**, **Budget**: the presets;
- **Customise**, when no team is saved (it would be a fifth option otherwise).

Each option's preview shows the whole team as a table, orchestrator included. **Other** accepts a saved team's name, or a change in plain words such as "last team, but Grok as worker".

Then show the confirm screen.

### Presets

Presets are rules, resolved against the catalog on every run:

- **Balanced**: advisor from the strongest family the workers do not use; workers scaled to difficulty; explorers the fastest cheap model; researchers mid-tier.
- **Max quality**: the strongest model for each role at high effort; explorers stay fast.
- **Budget**: the cheapest capable model for each role; the advisor stays a capable reasoner at medium effort, because it is the safety net.

## Screen 2: per role

Four questions on one screen: Advisor, Workers, Explorers, Researchers. Each has:

- **Suggested: `<model> · <effort>`** (recommended), following the role needs;
- **Last used: `<model> · <effort>`**, from the last confirmed team, when it differs from the suggestion; otherwise a second suggestion from another family;
- **Other**: the user types any model.

Model and effort form one option. The preview gives provider, model, effort, why it is suggested, and the estimated cost.

### Typed models

Resolve what the user types against the catalog:

- **Loose match**: name fragments plus an optional effort word (`opus high`, `gpt-6-sol xhigh`, `grok fast`).
- **Native provider first**: when the maker's own provider and an aggregator both serve the model, use the maker's.
- **Effort**: missing means the role's default effort; an effort the model lacks becomes the nearest one it offers.
- **No match, or several**: one follow-up question listing the candidates.
- **"show models"**: list the catalog compactly, grouped by provider, then ask again.

Then show the confirm screen.

## Screen 3: confirm

Show the goal card and a team table with every resolved model, including how typed text was matched (`opus high → Claude Opus 5.5 · high`). Add a warning line when:

- the advisor and the workers share a family: the code review is less independent;
- the advisor runs the orchestrator's model: the plan review shares its blind spots;
- a saved model left the catalog and was replaced.

Options:

- **Start**
- **Change roles**: go to screen 2, prefilled with this team.
- **Change goal**: the user corrects the goal card in a sentence; show this screen again.
- **Save as team**: ask for a name, save, then start.

Once the user chooses **Start** or **Save as team**, record the team as the last used and continue with step 5.

## Saved teams

`~/.orchestrate/teams.json`:

```json
{
  "version": 1,
  "last": "_last",
  "teams": {
    "_last": {
      "advisor":    { "provider": "<providerInstanceId>", "model": "<model id>", "effort": "high" },
      "worker":     { "provider": "<providerInstanceId>", "model": "<model id>", "effort": "medium" },
      "explorer":   { "provider": "<providerInstanceId>", "model": "<model id>", "effort": "low" },
      "researcher": { "provider": "<providerInstanceId>", "model": "<model id>", "effort": "medium" }
    }
  }
}
```

`last` names the team confirmed most recently; an unnamed confirmed team is stored as `_last`. Before offering a saved team, check each model against the catalog and replace any that is gone with the current suggestion for that role.
