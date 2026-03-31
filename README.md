# CLI Skill Test — Thin Skill Only

This repo tests whether an agent with ONLY the thin `alchemy-cli` skill (+ the CLI itself) can complete a battery of Alchemy tasks.

## Setup

1. Make sure `@alchemy/cli` is installed: `npm i -g @alchemy/cli`
2. Make sure `ALCHEMY_API_KEY` is set in your environment
3. Open this repo in Cursor (or your agent platform)
4. Start a new agent session

## Run the test

Tell the agent:

> Read TEST_PLAN.md and run every task in order. Record your results in results/results.md using the format specified in the test plan.

## What's in this repo

- `skills/alchemy-cli/SKILL.md` — the thin CLI skill (the ONLY Alchemy context the agent has)
- `.cursor/rules/alchemy.md` — tells the agent to use the skill
- `TEST_PLAN.md` — 27 tasks to complete
- `results/results.md` — where results get recorded
