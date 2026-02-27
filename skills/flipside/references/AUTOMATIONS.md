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
| `alert` | Evaluate conditions and fire alerts | `title`, `level`, `condition`, `blocks` |
| `report` | Generate a deterministic report | `title`, `blocks` (see [Report Builder vs Report Step](#report-builder-vs-report-step)) |
| `conditional` | Branch based on conditions | `condition` (natural language) |
| `upload` | Include uploaded CSV data | `uploadId` |
| `email` | Send email notification | `to`, `subject`, `body` |
| `slack` | Send Slack message (deterministic when downstream of alert) | `channel`, `message` |
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

### Alert Step

```yaml
- id: whale_alert
  type: alert
  config:
    title: "Large Transfer Detected"
    level: critical              # critical | warning | opportunity | info | twitter
    condition: "Fire when any transfer exceeds $10M"
    blocks:
      - id: details
        type: text
        title: "What happened?"
        rules: "Summarize the transfer details in 2-3 sentences"
```

**Alert levels:** critical (red), warning (amber), opportunity (green), info (indigo), twitter (dark card with X branding).

### Report Builder vs Report Step

**Use report_builder agent directly 90% of the time.** The report step is mainly useful in the web app when you need to deterministically control the report layout before a final alert/notification step. For most CLI workflows, just use an agent step with report_builder instead.

**Report step** (deterministic layout — you control exactly what panels appear):

```yaml
- id: build_report
  type: report
  config:
    title: "Weekly DEX Volume Analysis"
    description: "Overview of DEX activity across top protocols"
    blocks:
      - type: chart          # chart | table | metric
        chartType: line       # Optional: line, bar, area, donut, heatmap, treemap, bubble
        title: "Daily Volume Trend"
        intent: "Line chart showing daily DEX volume over the past 30 days"
      - type: metric
        title: "Total Volume"
        intent: "Sum of all DEX volume in the period, formatted as USD"
      - type: table
        title: "Top Protocols"
        intent: "Table of top 10 protocols by volume with protocol name and total volume"
    stylingHints: "Use a clean color palette, emphasize trends"
```

**Agent step with report_builder** (flexible — AI decides what to show):

```yaml
- id: build_report
  type: agent
  config:
    agentConfigId: flipside/report_builder
    instructions: "Create a report showing DEX volume trends and top protocols"
```

**Creating agents that generate reports:**

```yaml
# In your agent YAML
skills:
  - flipside/data_visualization   # Provides build_report, update_report, inspect_query_data
subAgents:
  - flipside/report_builder       # Delegates report generation
```

Supported block types: `chart`, `table`, `metric`. Do NOT use HTML blocks unless explicitly asked.

Deprecated patterns to avoid:
- `reporting` skill (replaced by `data_visualization`)
- `generate_report` tool (replaced by `build_report`)

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
