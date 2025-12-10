# Flipside CLI

Build and deploy AI agents for blockchain analytics. Query 40+ chains, analyze wallets, track DeFi protocols, and more.

## Quick Start

**1. Get an API key** from [flipsidecrypto.xyz](https://flipsidecrypto.xyz) → Settings → MCP Keys

**2. Configure the CLI**
```bash
./flipside config init
# Enter your API key when prompted
```

**3. Verify it works**
```bash
./flipside tools list
```

**4. Deploy your first agent**
```bash
./flipside agent push examples/defi_analyst.agent.yaml
./flipside agent run defi_analyst --message "What's the TVL on Aave?"
```

That's it! You now have a DeFi analyst agent ready to query blockchain data.

---

## Example Agents

This repo includes 7 ready-to-deploy agents in the [examples/](./examples/) directory:

| Agent | Type | What it does |
|-------|------|--------------|
| `defi_analyst` | chat | Analyze DeFi protocols - TVL, liquidity, DEX volume |
| `wallet_analyzer` | chat | Investigate wallet portfolios and transaction patterns |
| `swap_assistant` | chat | Execute cross-chain token swaps |
| `nft_analyst` | chat | Track NFT collections, sales, and holders |
| `protocol_researcher` | chat | Deep-dive protocol research with on-chain + web data |
| `tx_parser` | sub | Parse any transaction into structured JSON |
| `token_flow_analyzer` | sub | Analyze token movements over time |

### Deploy all examples at once
```bash
for f in examples/*.agent.yaml; do ./flipside agent push "$f"; done
```

### Try them out
```bash
# DeFi analysis
./flipside agent run defi_analyst --message "Top DEX protocols by volume this week on Arbitrum"

# Wallet investigation
./flipside agent run wallet_analyzer --message "Analyze vitalik.eth's recent activity"

# NFT market
./flipside agent run nft_analyst --message "What are the top NFT sales today?"

# Cross-chain swaps
./flipside agent run swap_assistant --message "Get a quote to swap 100 USDC from Ethereum to Base"

# Transaction parsing (sub agent - uses JSON input)
./flipside agent run tx_parser --data-json '{"tx_hash": "0x...", "chain": "ethereum"}'
```

See [examples/README.md](./examples/README.md) for detailed docs on each agent.

---

## Build Your Own Agent

### Create a new agent
```bash
./flipside agent init my_agent              # Chat agent (conversational)
./flipside agent init my_parser --kind sub  # Sub agent (structured I/O)
```

### Edit the YAML file
```yaml
name: my_agent
kind: chat
description: "What this agent does"

systemprompt: |
  You are an expert at... [customize this]

tools:
  - name: run_sql_query    # Query blockchain data
  - name: find_tables      # Discover available tables
  - name: search_web       # Web search for context

maxturns: 10
metadata:
  model: claude-sonnet-4   # or claude-4-5-haiku, claude-opus-4-5
```

### Deploy and run
```bash
./flipside agent validate my_agent.agent.yaml
./flipside agent push my_agent.agent.yaml
./flipside agent run my_agent --message "Hello!"
```

---

## Available Tools

Your agents can use these tools:

| Tool | Description |
|------|-------------|
| `run_sql_query` | Execute SQL against 40+ blockchain datasets |
| `find_tables` | Semantic search to discover relevant tables |
| `get_table_schema` | Get column details for a table |
| `search_web` | Search the web for context |
| `get_swap_quote` | Get cross-chain swap quotes |
| `execute_swap` | Execute a cross-chain swap |
| `get_swap_status` | Check swap status |
| `get_swap_tokens` | List available swap tokens |
| `find_workflow` | Find pre-built analysis workflows |
| `publish_html` | Publish visualizations to a public URL |

List all tools: `./flipside tools list`

---

## Run SQL Directly

Don't need an agent? Query data directly:

```bash
./flipside tools execute run_sql_query '{"query": "SELECT * FROM ethereum.core.fact_blocks LIMIT 5"}'
```

---

## Use Catalog Agents

Flipside maintains pre-built agents you can use immediately:

```bash
# List available agents
./flipside catalog agents list

# Run one
./flipside catalog agents run data_analyst --message "What's trending in DeFi?"
```

---

## Interactive Chat

Start a REPL for continuous conversation:

```bash
./flipside chat repl
```

---

## Command Reference

```bash
# Agents
./flipside agent init <name>           # Create new agent
./flipside agent validate <file>       # Validate YAML
./flipside agent push <file>           # Deploy agent
./flipside agent run <name> --message  # Run chat agent
./flipside agent run <name> --data-json # Run sub agent
./flipside agent list                  # List your agents
./flipside agent describe <name>       # View agent details
./flipside agent delete <name>         # Delete agent

# Catalog
./flipside catalog agents list         # List Flipside agents
./flipside catalog agents run <name>   # Run catalog agent

# Tools
./flipside tools list                  # List available tools
./flipside tools execute <tool> <json> # Execute a tool directly

# Config
./flipside config show                 # Show current config
./flipside config init                 # Setup wizard
```

### Global Flags

| Flag | Description |
|------|-------------|
| `-j, --json` | Output as JSON (for scripting) |
| `-v, --verbose` | Show request/response details |
| `--api-key` | Override API key for this command |

---

## Troubleshooting

**"API key not found"** → Run `./flipside config init`

**"Agent not found"** → Check `./flipside agent list` for your agents

**Validation errors** → Run `./flipside agent validate <file>` for details

**Debug mode** → Add `-v` flag to see full request/response

---

## Links

- [Flipside Docs](https://docs.flipsidecrypto.xyz)
- [Discord](https://discord.gg/flipside)
- [GitHub Issues](https://github.com/flipsidecrypto/flipside-cli/issues)
