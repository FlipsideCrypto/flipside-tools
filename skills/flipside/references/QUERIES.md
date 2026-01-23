# Queries & Table Discovery

## Finding Tables

Use `find_tables` to discover relevant tables by keyword:

```bash
flipside tools find_tables "token transfers"
flipside tools find_tables "dex swaps"
flipside tools find_tables "nft sales"
flipside tools find_tables "uniswap"
```

This searches table names and descriptions across all chains.

## Getting Table Schema

Once you find a table, get its schema:

```bash
flipside tools get_table_schema ethereum.core.ez_token_transfers
flipside tools get_table_schema solana.defi.fact_swaps
```

This shows all columns, data types, and descriptions.

## Table Naming Conventions

Flipside tables follow this pattern:

```
<chain>.<category>.<table_name>
```

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

```bash
# Prefer EZ tables for common queries
flipside tools get_table_schema ethereum.core.ez_token_transfers

# Use fact tables for advanced analysis
flipside tools get_table_schema ethereum.core.fact_token_transfers
```

## Running SQL

### Through Agents (Recommended)

Let agents write the SQL—they know the schema:

```bash
flipside agents run flipside/sql_agent --message "Get ETH transfers over 100 ETH in the last 24 hours"
```

### Direct Queries

For simple queries or when you know the exact SQL:

```bash
# Create and execute in one step
flipside query create "SELECT * FROM ethereum.core.ez_token_transfers LIMIT 10"

# Or manage saved queries
flipside query create --name "My Query" "SELECT ..."
flipside query list
flipside query execute <query-id>
```

## Query Results

### Getting Results

```bash
# Execute returns a run ID
flipside query execute <query-id>
# Output: Run ID: abc123...

# Get results
flipside query-run result <run-id>

# Poll until complete
flipside query-run poll <run-id>

# Download as CSV
flipside query-run download <run-id> --format csv
```

### Result Formats

```bash
flipside query-run result <run-id>              # Default table format
flipside query-run result <run-id> --json       # JSON output
flipside query-run download <run-id> -o out.csv # Save to file
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
4. **Check column names** with `get_table_schema` before writing queries
5. **Let agents help** when you're unsure about table structure
