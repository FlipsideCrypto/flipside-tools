# Automations

Automations are intelligent workflows that orchestrate SQL queries, AI transforms, and multi-agent systems in parallel. Execute on demand or schedule in advance, toggle notification cadence, and route results to Slack, Discord, Email, or Telegram. Trigger ad-hoc runs, chat with an agent to analyze results, and compare against past runs.

## Commands

```bash
flipside automations init my_automation       # Create automation YAML
flipside automations push <file>              # Deploy automation
flipside automations pull <id> -o <file>      # Export existing automation
flipside automations list                     # List automations
flipside automations get <id>                 # View automation details
flipside automations delete <id>              # Delete automation

flipside automations run <id>                 # Run automation
flipside automations run <id> -i '{"wallet_address":"0x..."}'  # Run with inputs

flipside automations runs list <automation-id>      # List runs
flipside automations runs get <run-id>              # Get run details
flipside automations runs result <run-id>           # Get full results
flipside automations runs result <run-id> --summary # Include AI summaries
flipside automations runs compare <id-a> <id-b>     # Compare two runs
flipside automations runs cancel <run-id>           # Cancel running automation

flipside chat --run <run-id>                        # Chat with an agent to analyze results
```

## Research Before Building

Use Flipside agents to discover tables, understand schemas, and prototype queries before building automations:

```bash
# Interactive chat - explore data conversationally
flipside chat

# Quick research queries via SQL agent
flipside agents run flipside/sql_agent --message "What tables have Uniswap swap data?"
flipside agents run flipside/sql_agent --message "Show me the schema for ethereum.defi.ez_dex_swaps"

# Discover tables programmatically
flipside tools find_tables "token transfers"

# Get exact column names
flipside tools get_table_schema ethereum.core.ez_token_transfers
```

**Tip:** Always verify table and column names before writing automation SQL. The agents know the schema and will write correct queries.

## YAML Structure

```yaml
name: DeFi Portfolio Report
description: Analyze wallet activity and send alerts
visibility: personal  # or organization

inputSchema:
  type: object
  properties:
    wallet_address:
      type: string
      title: Wallet Address
    chain:
      type: string
      enum: [ethereum, base, arbitrum]
      default: ethereum
  required: [wallet_address]

steps:
  - id: get_transfers
    name: Get Token Transfers
    type: sql_statement
    config:
      query: |
        SELECT symbol, SUM(amount_usd) as total
        FROM {{{chain}}}.core.ez_token_transfers
        WHERE to_address = {{wallet_address}}
        GROUP BY symbol
        ORDER BY total DESC
        LIMIT 10

  - id: analyze
    name: Analyze Results
    type: llm_transform
    config:
      prompt: "Summarize the key findings from this wallet activity"
      outputFormat: plain_text

  - id: notify
    name: Send Alert
    type: slack
    config:
      prompt: "Create a summary report of this analysis"

edges:
  - from: get_transfers
    to: analyze
  - from: analyze
    to: notify
```

## Step Types

| Type | Description |
|------|-------------|
| `sql_statement` | Execute SQL against 40+ chains |
| `sql_query_id` | Execute a saved query by ID |
| `agent` | Run an agent with its own skills and tools (see [Agent Steps](#agent-steps)) |
| `llm_transform` | Transform or summarize data with AI |
| `alert` | Evaluate conditions and fire alerts (see [Alert Steps](#alert-steps)) |
| `report` | Generate a deterministic report with specific panel layout (see [Report Builder vs Report Step](#report-builder-vs-report-step)) |
| `email` | Send email alerts |
| `slack` | Send Slack alerts (deterministic when downstream of alert) |
| `discord` | Send Discord alerts |
| `telegram` | Send Telegram alerts |
| `upload` | Pull in your own CSV data |
| `conditional` | Branch logic based on natural language conditions |

**Data flow:** Steps connected by edges automatically receive upstream outputs. No explicit variable references needed—LLM and agent steps see all incoming data.

## Input Variables

Reference inputs in SQL steps using:

- `{{param}}` — Auto-quoted (strings get quotes, numbers don't)
- `{{{param}}}` — Raw value (for table names, identifiers)

```yaml
config:
  query: |
    SELECT * FROM {{{chain}}}.core.ez_token_transfers
    WHERE to_address = {{wallet_address}}
```

## Conditional Branching

Route execution based on natural language conditions:

```yaml
steps:
  - id: check_volume
    name: Check if High Volume
    type: conditional
    config:
      condition: "Total transaction volume exceeds $1,000,000"

edges:
  - from: check_volume
    to: send_alert
    sourceHandle: "true"   # When condition is true
  - from: check_volume
    to: log_normal
    sourceHandle: "false"  # When condition is false
```

## Using Uploaded Files

Reference uploaded CSVs in automation steps:

```yaml
steps:
  - id: load_wallets
    type: upload
    config:
      uploadId: wallets.csv

  - id: query
    type: sql_statement
    config:
      query: |
        SELECT w.label, t.symbol, SUM(t.amount_usd) as total
        FROM ${upload:wallets.csv} w
        JOIN ethereum.core.ez_token_transfers t
          ON w.address = t.to_address
        GROUP BY w.label, t.symbol
```

## Agent Steps

Embed an agent directly in your automation:

```yaml
- id: research
  name: Research Wallet
  type: agent
  config:
    agentConfigId: flipside/sql_agent   # Find agents with: flipside agents list
    instructions: "Analyze this wallet's DeFi activity and summarize key positions"
```

The agent receives all upstream step outputs and workflow inputs as context.

## Alert Steps

Alert steps evaluate conditions against upstream data and fire structured alerts:

```yaml
- id: whale_alert
  name: Whale Movement Alert
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

**Twitter level:** When level is `twitter`, upstream data can include engagement metrics (likes, retweets, replies, views) and a tweet author handle. These render on the card automatically.

## Report Builder vs Report Step

**Use report_builder agent directly 90% of the time.** The report step is mainly useful in the web app when you need to deterministically control the report layout before a final alert/notification step. For most CLI workflows, just use an agent step with report_builder instead.

### Report Step (deterministic layout)

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
    stylingHints: "Use a clean color palette, emphasize trends"  # Optional
```

**Block types:** chart (visualization), table (data grid), metric (single KPI value).

### Agent Step with report_builder (flexible)

```yaml
- id: build_report
  type: agent
  config:
    agentConfigId: flipside/report_builder
    instructions: "Create a report showing DEX volume trends and top protocols"
```

The report_builder agent decides what panels to create based on the data. More flexible, better for exploratory analysis.

### Creating Agents That Generate Reports

When building custom agents that need to create reports, use the `data_visualization` skill:

```yaml
# In your agent YAML
skills:
  - flipside/data_visualization   # Provides build_report, update_report, inspect_query_data
subAgents:
  - flipside/report_builder       # Delegates report generation
```

**Supported block types:** chart, table, metric. Do NOT use HTML blocks unless the user explicitly asks for custom HTML output.

**Deprecated patterns to avoid:**
- `reporting` skill (replaced by `data_visualization`)
- `generate_report` tool (replaced by `build_report`)

## Deploying Updates

When you first push an automation, the CLI writes the assigned `id` back to your YAML file:

```yaml
id: abc-123-def  # Added after first push
name: My Automation
```

Subsequent pushes update the existing automation instead of creating a new one. To create a new automation from an existing file, remove the `id` field.

## Chat with Results

After an automation runs, you can chat with the results using AI:

```bash
# Start interactive chat about a specific run
flipside chat --run <run-id>
```

This opens an interactive session where you can:
- Ask questions about the run results
- Request analysis of step outputs
- Get AI-generated insights
- Explore the data conversationally

## Execution Model

Automations execute in **waves**:

1. The system builds a dependency graph from steps and edges
2. Steps with no dependencies run first (Wave 1)
3. Steps whose dependencies are complete run next (Wave 2)
4. This continues until all steps complete

Steps within the same wave execute in parallel for efficiency.

## Example: Multi-Step Analysis Pipeline

```yaml
name: Weekly DEX Report
description: Analyze DEX activity and notify team

inputSchema:
  type: object
  properties:
    lookback_days:
      type: integer
      default: 7
  required: []

steps:
  # Wave 1: Parallel queries
  - id: top_pools
    name: Top Pools by Volume
    type: sql_statement
    config:
      query: |
        SELECT pool_name, SUM(amount_usd) as volume
        FROM ethereum.defi.ez_dex_swaps
        WHERE block_timestamp >= CURRENT_DATE - {{lookback_days}}
        GROUP BY pool_name
        ORDER BY volume DESC
        LIMIT 20

  - id: top_traders
    name: Top Traders
    type: sql_statement
    config:
      query: |
        SELECT origin_from_address, COUNT(*) as swaps
        FROM ethereum.defi.ez_dex_swaps
        WHERE block_timestamp >= CURRENT_DATE - {{lookback_days}}
        GROUP BY origin_from_address
        ORDER BY swaps DESC
        LIMIT 20

  # Wave 2: Analyze results
  - id: analyze
    name: Generate Report
    type: llm_transform
    config:
      prompt: |
        Create a weekly DEX report with:
        - Top pools by volume
        - Most active traders
        - Key trends and insights
      outputFormat: markdown

  # Wave 3: Notify
  - id: slack_notify
    name: Post to Slack
    type: slack
    config:
      prompt: "Format this report for Slack with clear sections"

edges:
  - from: top_pools
    to: analyze
  - from: top_traders
    to: analyze
  - from: analyze
    to: slack_notify
```

Run it:

```bash
flipside automations push weekly_dex_report.workflow.yaml
flipside automations run <automation-id> -i '{"lookback_days": 7}'
```

Monitor and explore results:

```bash
flipside automations runs list <automation-id>
flipside automations runs result <run-id>
flipside chat --run <run-id>
```

## Troubleshooting

**Validate before deploying:**
```bash
flipside automations validate my_automation.workflow.yaml
```

**Check run status and step-level errors:**
```bash
flipside automations runs get <run-id>      # Shows run state and step states
flipside automations runs result <run-id>   # Shows full output including errors
```

| Issue | Solution |
|-------|----------|
| `step id is required` | Every step needs a unique `id` field |
| `edge references unknown step` | Check that `from` and `to` in edges match step `id` values |
| `invalid step type` | Use valid types: `sql_statement`, `agent`, `llm_transform`, `alert`, `report`, etc. |
| `agent step requires agentConfigId` | Find valid agents with `flipside agents list` |
| Run stuck in `running` | Check `runs get` for step-level state; cancel with `runs cancel` |
| SQL errors | Verify table names with `flipside tools find_tables` |
