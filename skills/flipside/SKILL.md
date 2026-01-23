---
name: flipside
description: |
  Query blockchain data across 40+ chains, create AI agents for crypto analytics,
  and build automated data pipelines. Use when working with blockchain data,
  crypto wallets, DeFi protocols, NFTs, token transfers, or on-chain analytics.
  Requires the Flipside CLI (https://docs.flipsidecrypto.xyz/get-started/cli).
compatibility: Requires flipside CLI installed and authenticated (flipside login)
allowed-tools: Bash(flipside:*) Read Write
metadata:
  author: flipside
  version: "1.0"
  homepage: https://flipsidecrypto.xyz
---

# Flipside CLI

Query blockchain data, create AI agents, and build automated data pipelines.

## Golden Rule

**Always use Flipside agents for SQL.** Don't write SQL from scratch—use an agent to generate correct queries against Flipside's data warehouse.

```bash
# Chat agent (conversational) - use --message
flipside agents run flipside/sql_agent --message "Get the top 10 DEX swaps on Ethereum today"

# Sub agent (structured I/O) - use --data-json
flipside agents run flipside/token_analyzer --data-json '{"chain": "ethereum", "token_address": "0x..."}'

# Always use --title when testing (makes runs easier to find)
flipside agents run flipside/my_agent --message "test query" --title "Testing from Cursor"
```

Agents understand the schema, know the right tables, and write correct SQL. They have access to tools like `find_tables`, `get_table_schema`, and `run_sql_query`.

## Quick Start

```bash
# 1. Check authentication
flipside whoami

# 2. Start an interactive chat with the SQL agent
flipside chat

# 3. Or run a one-off query through an agent
flipside agents run flipside/sql_agent --message "Show me the largest ETH transfers today"
```

## Core Workflows

| Task | Command | Reference |
|------|---------|-----------|
| Explore data interactively | `flipside chat` | - |
| Run SQL through an agent | `flipside agents run flipside/sql_agent --message "..."` | [AGENTS.md](references/AGENTS.md) |
| Discover tables | `flipside tools find_tables "token transfers"` | [QUERIES.md](references/QUERIES.md) |
| Get table schema | `flipside tools get_table_schema ethereum.core.ez_token_transfers` | [QUERIES.md](references/QUERIES.md) |
| Create your own agent | `flipside agents init my_agent` | [AGENTS.md](references/AGENTS.md) |
| Build a data pipeline | `flipside automations init my_pipeline` | [AUTOMATIONS.md](references/AUTOMATIONS.md) |
| Create reusable skills | `flipside skills init my_skill` | [SKILLS.md](references/SKILLS.md) |

## Agent References

**IMPORTANT:** Agent references MUST use the `org/agent_name` format.

```bash
# Correct
flipside agents run flipside/sql_agent --message "..."

# Wrong - will fail
flipside agents run sql_agent --message "..."
```

## Command Cheatsheet

### Agents

```bash
flipside agents list                              # List available agents
flipside agents init my_agent                     # Create agent YAML
flipside agents validate my_agent.agent.yaml      # Validate before deploy
flipside agents push my_agent.agent.yaml          # Deploy agent
flipside agents run org/agent --message "..."     # Run chat agent
flipside agents run org/agent --data-json '{}'    # Run sub agent
flipside agents describe org/agent                # Show agent details
flipside agents pull org/agent                    # Download agent YAML
```

### Queries & Data

```bash
flipside tools find_tables "swap"                 # Find tables by keyword
flipside tools get_table_schema <table>           # Get column definitions
flipside query create "SELECT ..."                # Create saved query
flipside query execute <query-id>                 # Run saved query
flipside query-run result <run-id>                # Get query results
```

### Automations

```bash
flipside automations init my_pipeline             # Create automation YAML
flipside automations validate <file>              # Validate before deploy
flipside automations push <file>                  # Deploy automation
flipside automations run <id>                     # Run automation
flipside automations run <id> -i '{"key":"val"}'  # Run with inputs
flipside automations runs list <automation-id>    # List runs
flipside automations runs result <run-id>         # Get run results
```

### Skills

```bash
flipside skills list                              # List available skills
flipside skills init my_skill                     # Create skill YAML
flipside skills push my_skill.skill.yaml          # Deploy skill
```

### Chat

```bash
flipside chat                                     # Start interactive chat
flipside chat --run <run-id>                      # Chat about a run
flipside chat resume                              # Resume previous chat
flipside chat send-message --session-id <id> "x"  # Programmatic message
```

## When to Load References

Load the detailed reference files when you need:

- **[QUERIES.md](references/QUERIES.md)** - Table discovery, SQL patterns, result handling
- **[AGENTS.md](references/AGENTS.md)** - Agent YAML schema, chat vs sub agents, deployment
- **[AUTOMATIONS.md](references/AUTOMATIONS.md)** - Pipeline steps, DAG edges, scheduling
- **[SKILLS.md](references/SKILLS.md)** - Creating reusable tool bundles
- **[TABLES.md](references/TABLES.md)** - Common tables by chain (Ethereum, Solana, etc.)

## Validation Before Deploy

Always validate YAML files before pushing:

```bash
# Validate agent
flipside agents validate my_agent.agent.yaml

# Validate automation
flipside automations validate my_pipeline.yaml
```

Or use the included scripts:

```bash
scripts/validate-agent.sh my_agent.agent.yaml
scripts/validate-automation.sh my_pipeline.yaml
scripts/check-auth.sh
```

## Templates

Starter templates are available in `assets/`:

- `assets/agent.template.yaml` - Basic agent config
- `assets/automation.template.yaml` - Basic automation config
- `assets/skill.template.yaml` - Basic skill config

Copy and customize these for new projects.
