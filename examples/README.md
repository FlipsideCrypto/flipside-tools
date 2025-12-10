# Example Agents

Two example agents demonstrating chat and sub agent types.

## defi_analyst (chat)

Conversational agent for DeFi protocol analysis.

```bash
# Deploy
./flipside agent push examples/defi_analyst.agent.yaml

# Run
./flipside agent run defi_analyst --message "What's the TVL on Aave?"
./flipside agent run defi_analyst --message "Compare DEX volume on Arbitrum vs Base"
```

**Tools**: `run_sql_query`, `find_tables`, `get_table_schema`, `search_web`

---

## tx_parser (sub)

Programmatic agent that parses transactions into structured JSON. Use for automation and pipelines.

```bash
# Deploy
./flipside agent push examples/tx_parser.agent.yaml

# Run with JSON input
./flipside agent run tx_parser --data-json '{"tx_hash": "0x...", "chain": "ethereum"}'
```

**Input**:
```json
{
  "tx_hash": "0x...",
  "chain": "ethereum"
}
```

**Output**: Structured summary with tx type, addresses, value, and human-readable explanation.
