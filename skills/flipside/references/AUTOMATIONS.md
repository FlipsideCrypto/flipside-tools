# Automations (Data Pipelines)

Automations are intelligent workflows that orchestrate SQL queries, AI transforms, and multi-agent systems. Execute on demand or schedule in advance, and route results to Slack, Discord, Email, or Telegram.

## Research First

Before building automations, use agents to discover tables and prototype queries:

```bash
# Interactive exploration
flipside chat

# Quick research
flipside agents run flipside/sql_agent --message "What tables have Uniswap swap data?"
flipside agents run flipside/sql_agent --message "Show me the schema for ethereum.defi.ez_dex_swaps"

# Programmatic discovery
flipside tools find_tables "token transfers"
flipside tools get_table_schema ethereum.core.ez_token_transfers
```

## Creating an Automation

### 1. Initialize

```bash
flipside automations init my_pipeline
# Creates: my_pipeline.automation.yaml
```

### 2. Define Steps and Edges

Edit the YAML to define your workflow (see schema below).

### 3. Validate

```bash
flipside automations validate my_pipeline.automation.yaml
```

### 4. Deploy

```bash
flipside automations push my_pipeline.automation.yaml
# Output: Deployed as automation ID: abc123...
```

The CLI writes the `id` back to your YAML. Subsequent pushes update the same automation.

### 5. Run

```bash
flipside automations run <automation-id>
flipside automations run <automation-id> -i '{"wallet_address": "0x..."}'
```

## YAML Schema

```yaml
name: My Automation
description: What this automation does
visibility: personal              # personal or organization

# Input parameters
inputSchema:
  type: object
  properties:
    wallet_address:
      type: string
      title: Wallet Address
    days_back:
      type: integer
      default: 7
  required: [wallet_address]

# Workflow steps
steps:
  - id: query_transfers
    name: Query Transfers
    type: sql_statement
    config:
      query: |
        SELECT * FROM ethereum.core.ez_token_transfers
        WHERE to_address = {{wallet_address}}
          AND block_timestamp >= CURRENT_DATE - {{days_back}}
        LIMIT 1000

  - id: analyze
    name: Analyze Results
    type: llm_transform
    config:
      prompt: "Summarize the key patterns in this wallet's transfer activity"
      outputFormat: plain_text

  - id: notify
    name: Send Alert
    type: slack
    config:
      channel: "#alerts"
      message: "Wallet analysis complete"

# DAG edges (data flow)
edges:
  - from: query_transfers
    to: analyze
  - from: analyze
    to: notify
```

## Step Types

| Type | Description | Config |
|------|-------------|--------|
| `sql_statement` | Execute SQL query | `query` |
| `sql_query_id` | Run saved query by ID | `queryId` |
| `agent` | Run a Flipside agent | `agentConfigId`, `instructions` |
| `llm_transform` | AI-powered data transformation | `prompt`, `outputFormat` |
| `conditional` | Branch based on conditions | `condition` (natural language) |
| `upload` | Include uploaded CSV data | `uploadId` |
| `email` | Send email notification | `to`, `subject`, `body` |
| `slack` | Send Slack message | `channel`, `message` |
| `discord` | Send Discord message | `webhookUrl`, `message` |
| `telegram` | Send Telegram message | `chatId`, `message` |

### SQL Statement Step

```yaml
- id: get_swaps
  type: sql_statement
  config:
    query: |
      SELECT * FROM ethereum.defi.ez_dex_swaps
      WHERE block_timestamp >= CURRENT_DATE - 1
      LIMIT 100
```

### Agent Step

```yaml
- id: research
  type: agent
  config:
    agentConfigId: flipside/sql_agent
    instructions: "Analyze this wallet's DeFi activity and identify patterns"
```

### LLM Transform Step

```yaml
- id: summarize
  type: llm_transform
  config:
    prompt: "Create a bullet-point summary of the key findings"
    outputFormat: plain_text  # or json, markdown
```

### Conditional Step

```yaml
- id: check_volume
  type: conditional
  config:
    condition: "If total volume exceeds $1M"
  # Edges from conditional steps can have conditions
```

## Data Flow

Steps connected by edges automatically receive upstream outputs. No explicit variable references needed—LLM and agent steps see all incoming data.

```yaml
edges:
  - from: query_data      # Output flows to analyze
    to: analyze
  - from: analyze         # Output flows to notify
    to: notify
```

## Input Variables

Use template syntax in SQL queries:

- `{{param}}` — Auto-quoted (strings get quotes, numbers don't)
- `{{{param}}}` — Raw value (for table names, identifiers)

```yaml
config:
  query: |
    SELECT * FROM {{{table_name}}}    -- Raw: ethereum.core.ez_token_transfers
    WHERE address = {{wallet}}         -- Quoted: '0x...'
      AND amount > {{min_amount}}      -- Unquoted: 1000
```

## Managing Automations

```bash
# List automations
flipside automations list

# Run automation
flipside automations run <automation-id>
flipside automations run <automation-id> -i '{"key": "value"}'

# List runs
flipside automations runs list <automation-id>

# Get run details
flipside automations runs get <run-id>

# Get full results
flipside automations runs result <run-id>

# Compare two runs
flipside automations runs compare <run-id-a> <run-id-b>

# Chat about results
flipside chat --run <run-id>
```

## Troubleshooting

```bash
# Validate before deploying
flipside automations validate my_pipeline.automation.yaml

# Check step-level errors
flipside automations runs get <run-id>

# View detailed logs
flipside automations runs get <run-id> --verbose
```

## Best Practices

1. **Research tables first** - Use agents to discover schema before writing SQL
2. **Validate before push** - Catches YAML and SQL errors early
3. **Start simple** - Build one step at a time, test, then add more
4. **Use meaningful step IDs** - `query_transfers` not `step1`
5. **Add descriptions** - Help future you understand the pipeline
6. **Test with small data** - Use LIMIT during development
