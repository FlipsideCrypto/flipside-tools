# Common Tables by Chain

Quick reference for the most commonly used tables across chains.

## Ethereum

### Core

| Table | Description |
|-------|-------------|
| `ethereum.core.ez_token_transfers` | ERC-20 token transfers with symbols and amounts |
| `ethereum.core.fact_transactions` | All transactions |
| `ethereum.core.fact_blocks` | Block data |
| `ethereum.core.fact_event_logs` | Contract event logs |
| `ethereum.core.dim_labels` | Address labels (exchanges, protocols, etc.) |

### DeFi

| Table | Description |
|-------|-------------|
| `ethereum.defi.ez_dex_swaps` | DEX swaps across all protocols |
| `ethereum.defi.ez_lending_borrows` | Lending protocol borrows |
| `ethereum.defi.ez_lending_repayments` | Lending repayments |
| `ethereum.defi.ez_lending_liquidations` | Liquidation events |
| `ethereum.defi.ez_bridge_activity` | Cross-chain bridge transactions |

### NFT

| Table | Description |
|-------|-------------|
| `ethereum.nft.ez_nft_sales` | NFT sales with prices |
| `ethereum.nft.ez_nft_transfers` | NFT transfers |
| `ethereum.nft.ez_nft_mints` | NFT minting events |

### Price

| Table | Description |
|-------|-------------|
| `ethereum.price.ez_prices_hourly` | Hourly token prices |
| `ethereum.price.dim_asset_metadata` | Token metadata |

## Solana

### Core

| Table | Description |
|-------|-------------|
| `solana.core.fact_transactions` | All transactions |
| `solana.core.fact_transfers` | SOL and SPL token transfers |
| `solana.core.fact_blocks` | Block data |
| `solana.core.dim_labels` | Address labels |

### DeFi

| Table | Description |
|-------|-------------|
| `solana.defi.fact_swaps` | DEX swaps |
| `solana.defi.ez_dex_swaps` | Curated DEX swaps |

### NFT

| Table | Description |
|-------|-------------|
| `solana.nft.fact_nft_sales` | NFT sales |
| `solana.nft.fact_nft_mints` | NFT mints |

## Polygon

### Core

| Table | Description |
|-------|-------------|
| `polygon.core.ez_token_transfers` | Token transfers |
| `polygon.core.fact_transactions` | All transactions |
| `polygon.core.dim_labels` | Address labels |

### DeFi

| Table | Description |
|-------|-------------|
| `polygon.defi.ez_dex_swaps` | DEX swaps |
| `polygon.defi.ez_lending_borrows` | Lending borrows |

## Arbitrum

### Core

| Table | Description |
|-------|-------------|
| `arbitrum.core.ez_token_transfers` | Token transfers |
| `arbitrum.core.fact_transactions` | All transactions |

### DeFi

| Table | Description |
|-------|-------------|
| `arbitrum.defi.ez_dex_swaps` | DEX swaps |

## Optimism

### Core

| Table | Description |
|-------|-------------|
| `optimism.core.ez_token_transfers` | Token transfers |
| `optimism.core.fact_transactions` | All transactions |

### DeFi

| Table | Description |
|-------|-------------|
| `optimism.defi.ez_dex_swaps` | DEX swaps |

## Base

### Core

| Table | Description |
|-------|-------------|
| `base.core.ez_token_transfers` | Token transfers |
| `base.core.fact_transactions` | All transactions |

### DeFi

| Table | Description |
|-------|-------------|
| `base.defi.ez_dex_swaps` | DEX swaps |

## Avalanche

### Core

| Table | Description |
|-------|-------------|
| `avalanche.core.ez_token_transfers` | Token transfers |
| `avalanche.core.fact_transactions` | All transactions |

### DeFi

| Table | Description |
|-------|-------------|
| `avalanche.defi.ez_dex_swaps` | DEX swaps |

## BSC (BNB Chain)

### Core

| Table | Description |
|-------|-------------|
| `bsc.core.ez_token_transfers` | BEP-20 transfers |
| `bsc.core.fact_transactions` | All transactions |

### DeFi

| Table | Description |
|-------|-------------|
| `bsc.defi.ez_dex_swaps` | DEX swaps |

## Cross-Chain

### Prices

| Table | Description |
|-------|-------------|
| `crosschain.price.ez_prices_hourly` | Prices across all chains |
| `crosschain.core.dim_date_times` | Date/time dimension table |

## Finding More Tables

Not all tables are listed here. Use these commands to discover more:

```bash
# Search by keyword
flipside tools find_tables "uniswap"
flipside tools find_tables "aave"
flipside tools find_tables "nft sales"

# Get schema for any table
flipside tools get_table_schema ethereum.defi.ez_dex_swaps
```

## Common Patterns

### Cross-Chain Query

```sql
SELECT 'ethereum' as chain, * FROM ethereum.defi.ez_dex_swaps WHERE ...
UNION ALL
SELECT 'polygon' as chain, * FROM polygon.defi.ez_dex_swaps WHERE ...
UNION ALL
SELECT 'arbitrum' as chain, * FROM arbitrum.defi.ez_dex_swaps WHERE ...
```

### Address Lookup with Labels

```sql
SELECT
  t.*,
  l.label_type,
  l.label
FROM ethereum.core.ez_token_transfers t
LEFT JOIN ethereum.core.dim_labels l
  ON t.from_address = l.address
WHERE t.block_timestamp >= CURRENT_DATE - 1
```

### Token Price Join

```sql
SELECT
  t.*,
  p.price * t.amount as value_usd
FROM ethereum.core.ez_token_transfers t
LEFT JOIN ethereum.price.ez_prices_hourly p
  ON t.symbol = p.symbol
  AND DATE_TRUNC('hour', t.block_timestamp) = p.hour
```
