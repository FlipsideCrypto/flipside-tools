# Example Agents

This directory contains example agent configurations demonstrating different use cases for Flipside AI agents.

## Quick Start

```bash
# 1. Pick an agent and validate it
flipside agent validate examples/defi_analyst.agent.yaml

# 2. Deploy it
flipside agent push examples/defi_analyst.agent.yaml

# 3. Run it
flipside agent run defi_analyst --message "What are the top DEX protocols by volume this week?"
```

## Chat Agents

Chat agents are conversational and accept natural language messages.

### defi_analyst.agent.yaml

**Purpose**: DeFi protocol analysis - TVL, liquidity pools, yield farming, DEX analytics

**Tools**: `run_sql_query`, `find_tables`, `get_table_schema`, `search_web`

**Example queries**:
```bash
flipside agent run defi_analyst --message "What's the TVL trend for Aave on Ethereum over the past month?"
flipside agent run defi_analyst --message "Compare DEX volume on Arbitrum vs Base this week"
flipside agent run defi_analyst --message "Which lending protocols have the highest utilization rates?"
```

---

### wallet_analyzer.agent.yaml

**Purpose**: Wallet activity analysis - portfolios, transaction history, behavior patterns

**Tools**: `run_sql_query`, `find_tables`, `get_table_schema`, `search_web`

**Example queries**:
```bash
flipside agent run wallet_analyzer --message "Analyze this wallet's activity: 0x..."
flipside agent run wallet_analyzer --message "What tokens does vitalik.eth hold?"
flipside agent run wallet_analyzer --message "Show me the largest ETH transfers in the last 24 hours"
```

---

### swap_assistant.agent.yaml

**Purpose**: Cross-chain token swaps - quotes, execution, and status tracking

**Tools**: `get_swap_quote`, `execute_swap`, `get_swap_status`, `get_swap_tokens`

**Example queries**:
```bash
flipside agent run swap_assistant --message "What tokens can I swap from Solana to Base?"
flipside agent run swap_assistant --message "Get a quote for swapping 100 USDC from Ethereum to Arbitrum"
flipside agent run swap_assistant --message "Check the status of my swap to address 0x..."
```

---

### nft_analyst.agent.yaml

**Purpose**: NFT market analysis - collections, sales, holders, marketplace activity

**Tools**: `run_sql_query`, `find_tables`, `get_table_schema`, `search_web`

**Example queries**:
```bash
flipside agent run nft_analyst --message "What are the top NFT collections by volume on Ethereum today?"
flipside agent run nft_analyst --message "Analyze the holder distribution for Pudgy Penguins"
flipside agent run nft_analyst --message "Show me the largest NFT sales this week"
```

---

### protocol_researcher.agent.yaml

**Purpose**: Comprehensive protocol research combining on-chain data with web research

**Tools**: `run_sql_query`, `find_tables`, `get_table_schema`, `search_web`, `find_workflow`

**Example queries**:
```bash
flipside agent run protocol_researcher --message "Give me an overview of the Uniswap protocol"
flipside agent run protocol_researcher --message "Research the GMX perpetuals protocol on Arbitrum"
flipside agent run protocol_researcher --message "Compare Lido vs Rocket Pool staking"
```

---

## Sub Agents

Sub agents are programmatic and accept structured JSON input. They're designed for automation and pipelines.

### tx_parser.agent.yaml

**Purpose**: Parse and explain blockchain transactions

**Tools**: `run_sql_query`, `find_tables`, `get_table_schema`

**Input Schema**:
```json
{
  "tx_hash": "0x...",
  "chain": "ethereum"
}
```

**Example usage**:
```bash
flipside agent run tx_parser --data-json '{"tx_hash": "0x123abc...", "chain": "ethereum"}'
```

**Output**: Structured transaction summary including type, parties, value, and human-readable explanation.

---

### token_flow_analyzer.agent.yaml

**Purpose**: Analyze token flows between addresses over a time period

**Tools**: `run_sql_query`, `find_tables`, `get_table_schema`

**Input Schema**:
```json
{
  "token_address": "0x...",
  "chain": "ethereum",
  "start_date": "2024-01-01",
  "end_date": "2024-01-31",
  "min_amount_usd": 1000
}
```

**Example usage**:
```bash
flipside agent run token_flow_analyzer --data-json '{
  "token_address": "0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48",
  "chain": "ethereum",
  "start_date": "2024-12-01",
  "end_date": "2024-12-07"
}'
```

**Output**: Structured flow data including top senders, receivers, and net flows.

---

## Customization Tips

### Changing the Model

Models available:
- `claude-4-5-haiku` - Fast and cost-effective (default for sub agents)
- `claude-sonnet-4` - Balanced performance (default for chat agents)
- `claude-opus-4-5` - Most capable

```yaml
metadata:
  model: claude-opus-4-5
```

### Adding Tools

Available tools:
- `run_sql_query` - Execute SQL against Flipside data
- `find_tables` - Semantic search for tables
- `get_table_schema` - Get table column details
- `search_web` - Web search for context
- `get_swap_quote` / `execute_swap` / `get_swap_status` / `get_swap_tokens` - Cross-chain swaps
- `find_workflow` / `get_workflow` / `plan_workflow` - Use pre-built workflows
- `publish_html` - Publish visualizations

```yaml
tools:
  - name: run_sql_query
  - name: search_web
```

### Composing Agents

Use subagents to delegate tasks:

```yaml
subagents:
  - tx_parser
  - token_flow_analyzer
```

### Setting Status

- `draft` - Development/testing
- `active` - Production ready
- `inactive` - Disabled

```yaml
status: active
```

## Deploying Multiple Agents

```bash
# Validate all
for f in examples/*.agent.yaml; do flipside agent validate "$f"; done

# Deploy all
for f in examples/*.agent.yaml; do flipside agent push "$f"; done
```

## Need Help?

- [Flipside Documentation](https://docs.flipsidecrypto.xyz)
- [Discord Community](https://discord.gg/flipside)
