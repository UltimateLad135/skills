# Model tiers

Shared by `/playtest`, `/council` and `/implement`. Each skill ships an identical copy of this file. If `~/.agents/tiers.md` exists, it replaces this file for every skill: change your models there once, instead of in each installed copy.

| Tier | Model | Provider | Model id | Option |
|---|---|---|---|---|
| **swarm**: cheap, run many | DeepSeek V4.1 Flash | `opencode` | `opencode-go/deepseek-v4.1-flash` | variant `max` |
| | Gemini 3.8 Flash | `antigravity` | `gemini-3.8-flash` | thinking_level `high` |
| | Claude Haiku 5.5 (reserve) | `claudeAgent` | `claude-haiku-5-5` | effort `max` |
| | GPT-6 Luna (reserve) | `codex` | `gpt-6-luna` | reasoningEffort `max` |
| **mid**: smarter, a few at a time | Claude Sonnet 5.5 | `claudeAgent` | `claude-sonnet-5-5` | effort `high` |
| | GPT-6.1 Terra | `codex` | `gpt-6.1-terra` | reasoningEffort `high` |
| **review**: strongest, spend carefully | Claude Opus 5.5 | `claudeAgent` | `claude-opus-5-5` | effort `high` |
| | GPT-6.1 Sol | `codex` | `gpt-6.1-sol` | reasoningEffort `xhigh` |
| | Gemini 4 Argon | `antigravity` | `gemini-4-argon` | thinking_level `high` |
| | Grok 4.7 | `grok` | `grok-4.7` | reasoningEffort `xhigh` |

## Rules

- Each skill says which tier a job uses and how many agents it gets. Table order is preference order: "the first model" of a tier means the first one that is available.
- **Reserves** run only when every model above them in their tier is missing or has failed. A job that uses "all models" of a tier uses only its models that aren't reserves. Haiku and Luna are reserves because they draw on the same usage limits as the bigger Claude and GPT models; the swarm's work belongs on DeepSeek, which doesn't.
- Call `orchestrator_capabilities` first, and send only the options it lists for a model.
- Models on your own provider may run as native subagents (Claude Code's `Agent` with its `model` parameter when you run on Claude). Every other model goes through `delegate_task`.
- A missing model is skipped: say which, and spread its jobs over the rest of its tier. If a whole tier is missing, say so and use the next tier up. Never move a job down a tier, and never use a model that isn't in the table.
- When a provider renames a model, update the table: every copy in the skills repository, or `~/.agents/tiers.md`.
