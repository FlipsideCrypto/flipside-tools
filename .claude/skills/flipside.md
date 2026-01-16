# Flipside CLI

Query blockchain data across 40+ chains via SQL and AI agents.

## Before You Start

Run this command to get full usage instructions:

```bash
flipside llm onboard
```

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

## Key Rules

### Always Discover Agents First

1. **If a user mentions a specific agent**: Always run `flipside agents list` first to find the correct agent name. Agent names may differ from what the user says (e.g., "monad validator agent" might be `flipside/monad_staking_agent`).

2. **If a user asks about a specific topic** (e.g., "forecast validator earnings on Monad", "analyze staking rewards"): Run `flipside agents list` first to check if there's a specialized agent for that domain before defaulting to `flipside/sql_agent`. Specialized agents have domain knowledge and produce better results.

```bash
# Always start by listing agents to find the right one
flipside agents list

# Then use the most relevant agent for the task
flipside agents run <correct_agent_name> --message "your query"
```

### Use Agents Over Raw SQL

For complex queries, use agents instead of writing SQL directly. Agents know the schema and write correct queries.
