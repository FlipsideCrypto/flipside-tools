# Queries & SQL Execution

## Running SQL Queries

### Through Chat (Recommended)

Use `flipside chat` for interactive data exploration. The chat interface has built-in agents that understand Flipside's schema:

```bash
flipside chat
```

Then ask questions like:
- "Show me the top DEX swaps on Ethereum today"
- "What are the largest USDC transfers in the last 24 hours?"
- "Find wallet activity for vitalik.eth"

### Through Agents

Run queries programmatically through your organization's agents:

```bash
# First, list available agents
flipside agents list

# Run a chat agent with your query request
flipside agents run <org>/<agent_name> --message "Get ETH transfers over 100 ETH in the last 24 hours"
```

### Direct SQL Queries

For simple queries or when you know the exact SQL:

```bash
# Create and execute a query
flipside query create "SELECT * FROM ethereum.core.ez_token_transfers LIMIT 10"

# Create a named saved query
flipside query create --name "My Query" "SELECT ..."

# List saved queries
flipside query list

# Execute a saved query by ID
flipside query execute <query-id>
```

## Query Results

### Getting Results

```bash
# Execute returns a run ID
flipside query execute <query-id>
# Output: Run ID: abc123...

# Check run status
flipside query-run status <run-id>

# Get results once complete
flipside query-run result <run-id>

# Poll until complete (blocks until done)
flipside query-run poll <run-id>
```

### Result Formats

```bash
flipside query-run result <run-id>              # Default table format
flipside query-run result <run-id> --json       # JSON output
```

## Table Naming Conventions

Flipside tables follow this pattern:

```
<chain>.<category>.<table_name>
```

### Supported Chains

Flipside supports 40+ chains including:
- ethereum, polygon, arbitrum, optimism, base, avalanche
- solana, near, flow, cosmos
- bitcoin, aptos, sui

### Categories

| Category | Description |
|----------|-------------|
| `core` | Core blockchain data (blocks, transactions, transfers) |
| `defi` | DeFi protocol data (swaps, liquidity, lending) |
| `nft` | NFT data (sales, mints, transfers) |
| `price` | Token prices and market data |
| `gov` | Governance and voting data |

### EZ Tables vs Raw Tables

- **EZ tables** (`ez_*`) - Curated, human-readable, recommended for most use cases
- **Fact tables** (`fact_*`) - Lower-level, more granular data
- **Dim tables** (`dim_*`) - Dimension/lookup tables

```sql
-- Prefer EZ tables for common queries
SELECT * FROM ethereum.core.ez_token_transfers LIMIT 10

-- Use fact tables for advanced analysis
SELECT * FROM ethereum.core.fact_token_transfers LIMIT 10
```

## Common Query Patterns

### Token Transfers

```sql
SELECT
  block_timestamp,
  from_address,
  to_address,
  symbol,
  amount
FROM ethereum.core.ez_token_transfers
WHERE block_timestamp >= CURRENT_DATE - 1
  AND symbol = 'USDC'
  AND amount > 1000000
ORDER BY amount DESC
LIMIT 100
```

### DEX Swaps

```sql
SELECT
  block_timestamp,
  platform,
  token_in,
  token_out,
  amount_in_usd
FROM ethereum.defi.ez_dex_swaps
WHERE block_timestamp >= CURRENT_DATE - 1
ORDER BY amount_in_usd DESC
LIMIT 100
```

### Wallet Activity

```sql
SELECT
  block_timestamp,
  tx_hash,
  from_address,
  to_address,
  eth_value
FROM ethereum.core.fact_transactions
WHERE from_address = LOWER('0x...')
   OR to_address = LOWER('0x...')
ORDER BY block_timestamp DESC
LIMIT 100
```

## Tips

1. **Always use LOWER()** for address comparisons—addresses are stored lowercase
2. **Use LIMIT** during development to avoid large result sets
3. **Filter by date** with `block_timestamp >= CURRENT_DATE - N`
4. **Use chat for discovery** - when unsure about tables, use `flipside chat` to explore
5. **Let agents help** when you're unsure about table structure or complex queries
