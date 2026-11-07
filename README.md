# Frontier Orchestrator

A Claude Code plugin that makes your expensive model (Fable or Opus) act as the **lead**. It hands searching, reading, test runs and routine edits to cheaper **Sonnet** helpers, then checks their work.

Pay senior rates for thinking, junior rates for reading.

## Why

Out of the box, Claude Code under Fable rarely hands work off on its own. When it does, the helper usually runs on an expensive model: Opus for the built-in search helper, or the main model for others. Most of a coding session is reading code, and that reading is billed at frontier prices.

This plugin changes the default:

- **A startup hook** puts short "delegate first" rules in front of the model at the start of every session.
- **Four helpers with their model fixed** in the file, so they can't fall back to an expensive one:
  - `scout`: Sonnet, read-only. Searching and reading. The default helper.
  - `builder`: Sonnet. Edits from a clear plan.
  - `checker`: Sonnet. Runs tests, lint and builds; never edits.
  - `deep-reviewer`: Opus, read-only. Only for security, concurrency, data integrity and migrations.
- **An `orchestrate` skill** with the full plan → discover → decide → build → verify → report playbook for big jobs.

The rules switch themselves off when the main model is Sonnet or Haiku.

## Install

In Claude Code:

```
/plugin marketplace add awalhossainofficial/frontier-orchestrator
/plugin install frontier-orchestrator@frontier-orchestrator
```

Start a new session so the startup rules load.

### Optional: lock every helper onto Sonnet

The plugin can't change settings, so these two steps are manual. They make helpers use Sonnet even when the model forgets to pick one.

1. Add this to `~/.claude/settings.json`. It needs Claude Code 2.1.251 or later; on older versions it overrides deliberate Opus choices too.
   ```json
   "env": { "CLAUDE_CODE_SUBAGENT_MODEL": "sonnet" }
   ```
2. Copy `plugins/frontier-orchestrator/extras/Explore.md` into `~/.claude/agents/`. It replaces Claude Code's built-in search helper with a Sonnet one. Anthropic's docs suggest doing exactly this.

## Results so far

All runs used Claude Code 2.1.319 on one machine and a real TypeScript app. Costs are Claude API list prices, as reported by Claude Code. These are single runs, so treat them as examples, not guarantees.

- **"Find every place this app calls an AI API"** (the prompt never mentions helpers):
  - Fable without the plugin: **$1.86**, no helpers.
  - Fable with the plugin: **$0.93**, with one Sonnet scout sent automatically.
  - Both answers cited 16 of the same files. The plugin run found 3 extra.
- **Codebase error-handling audit** (an earlier version of the same rules):
  - Without: **$5.02**.
  - With: **$3.06**, about 39% cheaper, finishing in 51 seconds instead of 4.5 minutes, with comparable findings.

## Good to know

- **Savings depend on the work.** Reading-heavy jobs save the most. For a tiny one-file fix, delegating can cost more than doing it directly, and the rules tell the model to skip helpers for those.
- **Helpers aren't always cheaper in tokens, only in dollars.** A run with helpers can use more tokens in total, because most of them are Sonnet tokens at about a fifth of the price.
- **Other providers behave differently.** On Amazon Bedrock, Google Cloud or Microsoft Foundry, Claude Code's defaults differ, so your baseline and savings will too.
- **Company settings win.** Organization-managed settings and model allowlists override this plugin.
- **Only tested on one machine and one account.** Reports from other setups are welcome as issues.

## How it works

- `hooks/hooks.json` runs `cat rules.md` at session start. Claude Code adds that output to the model's context.
- `agents/*.md` define the helpers. The `model:` line in each file is what Claude Code uses when the lead doesn't name a model.
- `skills/orchestrate/SKILL.md` is the long playbook, loaded only for big jobs.

Model selection follows Anthropic's documented order: the model named when a helper starts, then the helper file's `model:`, then `CLAUDE_CODE_SUBAGENT_MODEL`, then the main model. See https://code.claude.com/docs/en/sub-agents.

## Uninstall

```
/plugin uninstall frontier-orchestrator@frontier-orchestrator
```

If you did the optional steps, also remove the `env` line and `~/.claude/agents/Explore.md`.

## License

MIT
