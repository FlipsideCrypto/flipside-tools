# Flipside CLI

Query blockchain data across 40+ chains via SQL and AI agents.

## Before You Start

Run this command to get full usage instructions:

```bash
flipside llm onboard
```

Or load the skill: [docs/flipside.skill.md](https://github.com/FlipsideCrypto/flipside-tools/blob/main/docs/flipside.skill.md)

## Quick Reference

```bash
# Run SQL directly
flipside query create "SELECT * FROM ethereum.core.fact_blocks LIMIT 5"

# Use an agent (recommended for complex queries)
flipside agents run flipside/sql_agent --message "Top DEX swaps on Ethereum today"

# List available agents
flipside agents list

# Check your org
flipside whoami
```

## Key Rule

For complex queries, use agents instead of writing SQL directly. Agents know the schema and write correct queries.
