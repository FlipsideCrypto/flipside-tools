# Flipside CLI Skill

Query blockchain data across 40+ chains via SQL and AI agents.

## Setup

Run `flipside llm onboard` for full usage instructions.

## Quick Commands

```bash
# Run SQL
flipside query create "SELECT * FROM ethereum.core.fact_blocks LIMIT 5"

# Use an agent (recommended)
flipside agents run flipside/sql_agent --message "Top DEX swaps today"

# List agents
flipside agents list

# Check org
flipside whoami
```

## When to Use Agents vs SQL

**Use agents** for:
- Complex queries where you don't know the schema
- Natural language questions about blockchain data
- Any query involving joins or aggregations

**Use SQL directly** for:
- Simple queries where you know the exact table/columns
- Quick one-off lookups

## Key Pattern

```bash
# Always prefer agents for complex queries
flipside agents run flipside/sql_agent --message "Your question here"
```

Agents know the schema and write correct SQL. Don't guess at table names or columns.
