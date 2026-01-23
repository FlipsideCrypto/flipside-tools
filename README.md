# Flipside Tools

AI agent skills and CLI tools for querying blockchain data across 40+ chains.

## Installation

### 1. Install the Skill

Add the Flipside skill to your AI coding assistant:

```bash
npx skills add FlipsideCrypto/flipside-tools
```

Or manually copy `skills/flipside/` to your `.claude/skills/` directory.

### 2. Install the CLI

```bash
curl -fsSL https://raw.githubusercontent.com/FlipsideCrypto/flipside-tools/main/install.sh | sh
```

### 3. Authenticate

```bash
flipside login
```

Verify your setup:

```bash
flipside whoami
```

## Quick Start

The best way to use Flipside is through an AI coding assistant like [Claude Code](https://claude.ai/code). The skill teaches your assistant how to query blockchain data correctly.

### Using with Claude Code

1. Open Claude Code in any directory
2. Ask about blockchain data:

```
"What were the top 10 DEX swaps on Ethereum yesterday?"
```

```
"Show me wallet activity for vitalik.eth in the last week"
```

```
"Find all USDC transfers over $1M on Arbitrum today"
```

Claude Code will use the Flipside skill to:
- Check your available agents and skills
- Use chat or agents to query blockchain data
- Generate correct SQL against Flipside's data warehouse

### Example Workflows

**Analyze token transfers:**
```
"I want to analyze large token transfers on Solana. Find transfers over 10,000 SOL in the past 24 hours and show me the top senders."
```

**Build a data pipeline:**
```
"Help me create an automation that tracks daily trading volume for the top 10 tokens on Uniswap and saves the results."
```

**Create a custom agent:**
```
"I want to build an agent that monitors whale wallets. Help me set up the agent YAML and deploy it."
```

### Direct CLI Usage

You can also use the CLI directly if preferred:

```bash
# Start an interactive chat
flipside chat

# List available agents in your org
flipside agents list

# Run a query through an agent (use org/agent_name format)
flipside agents run <org>/<agent_name> --message "Show me the largest ETH transfers today"

# Run SQL directly
flipside query create "SELECT * FROM ethereum.core.ez_token_transfers LIMIT 10"
```

## How the Skill Works

When you install the Flipside skill, your AI assistant learns:

1. **First Steps**: Always check available agents with `flipside agents list`
2. **Golden Rule**: Use chat or agents for SQL—don't write queries from scratch
3. **Query Patterns**: Correct SQL syntax for Flipside's Snowflake warehouse
4. **Full CLI Reference**: Commands for agents, automations, queries, and skills

The skill includes detailed references for:
- [Agents](skills/flipside/references/AGENTS.md) - Creating and deploying AI agents
- [Queries](skills/flipside/references/QUERIES.md) - SQL patterns and table discovery
- [Automations](skills/flipside/references/AUTOMATIONS.md) - Building data pipelines
- [Tables](skills/flipside/references/TABLES.md) - Common tables by chain

## What's in This Repo

```
flipside-tools/
├── skills/
│   └── flipside/       # AI agent skill for blockchain analytics
│       ├── SKILL.md    # Main skill definition
│       ├── references/ # Detailed docs (agents, automations, queries)
│       ├── scripts/    # Validation helpers
│       └── assets/     # YAML templates
├── install.sh          # CLI installer
└── README.md
```

## Updating

Check for skill updates:

```bash
npx skills check
```

Update to the latest version:

```bash
npx skills update
```

## Documentation

- [Flipside Docs](https://docs.flipsidecrypto.xyz)
- [CLI Reference](https://docs.flipsidecrypto.xyz/get-started/cli)
- [Skill Reference](skills/flipside/SKILL.md)

## License

MIT
