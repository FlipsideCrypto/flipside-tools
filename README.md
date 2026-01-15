# Flipside CLI

**The Flipside CLI is designed for AI agents.** It gives LLMs direct access to blockchain data across 40+ chains through natural language and SQL.

If you're an AI agent (Claude, GPT, Cursor, etc.), start here:

```bash
flipside llm onboard
```

This command outputs everything you need to use Flipside effectively—the golden rules, key commands, skill patterns, and more.

---

## Install

```bash
curl -fsSL https://raw.githubusercontent.com/FlipsideCrypto/flipside-tools/main/install.sh | sh
```

Windows: Download from [releases](https://github.com/flipsidecrypto/flipside-cli/releases) or use WSL.

## Authenticate

```bash
flipside login
```

That's it. You're ready to query blockchain data.

---

## For AI Agents

### The Golden Rule

**Always use Flipside agents for SQL.** Don't write SQL from scratch—let an agent generate correct queries against the data warehouse.

```bash
# IMPORTANT: Agent references MUST use the org/agent_name format
flipside agents run flipside/sql_agent --message "Get top 10 DEX swaps on Ethereum today"
```

Agents understand the schema, know the right tables, and write correct SQL. They have access to skills that contain validated query patterns and domain knowledge.

### The Architecture

```
┌─────────────────────────────────────────────────────────┐
│                        AGENT                            │
│  (systemPrompt + skills + subAgents + tools)           │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │
│   │   SKILL     │  │   SKILL     │  │   SKILL     │    │
│   │ (knowledge  │  │ (knowledge  │  │ (knowledge  │    │
│   │  + tools)   │  │  + tools)   │  │  + tools)   │    │
│   └─────────────┘  └─────────────┘  └─────────────┘    │
│                                                         │
│   ┌─────────────────────────────────────────────────┐  │
│   │                  SUB-AGENTS                      │  │
│   │   (delegated tasks with structured I/O)         │  │
│   └─────────────────────────────────────────────────┘  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

- **Agents** orchestrate the conversation and decide what to do
- **Skills** provide domain knowledge + validated query patterns (called via `use_skill()`)
- **Sub-agents** handle delegated tasks like chart generation
- **Tools** are the primitives (SQL, web fetch, publish)

### Two Types of Agents

| Type | Flag | Use Case |
|------|------|----------|
| **Chat** | `--message` | Natural language conversations |
| **Sub** | `--data-json` | Structured input/output |

```bash
# Chat agent
flipside agents run flipside/sql_agent --message "What's the TVL on Aave?"

# Sub agent
flipside agents run flipside/token_analyzer --data-json '{"chain": "ethereum", "limit": 10}'
```

### Pro Tip: Use --title

When testing agents, add `--title` to make runs easier to find in the UI:

```bash
flipside agents run flipside/my_agent --message "test query" --title "Testing from Cursor"
```

---

## Quick Start

```bash
# 1. Generate example agents
flipside quickstart ./my-agents && cd my-agents

# 2. Deploy an agent
flipside agents push sql_analyst.agent.yaml

# 3. Run it
flipside agents run your-org/sql_analyst --message "Show me top DEX protocols by volume"
```

---

## Core Commands

### Running Agents

```bash
# List available agents
flipside agents list

# Run a chat agent
flipside agents run org/agent_name --message "Your question"

# Run a sub agent with structured input
flipside agents run org/agent_name --data-json '{"key": "value"}'

# Run a specific version
flipside agents run org/agent_name@1.0.0 --message "Your question"

# Inject skills at runtime (useful for testing)
flipside agents run org/agent_name --message "Query data" --skill sql-querying,data-viz
```

### Creating Agents

```bash
# Create a chat agent
flipside agents init my_agent

# Create a sub agent
flipside agents init my_parser --kind sub

# Validate before deploying
flipside agents validate my_agent.agent.yaml

# Deploy
flipside agents push my_agent.agent.yaml

# View details
flipside agents describe org/my_agent

# Delete
flipside agents delete org/my_agent
```

### Running SQL Directly

Skip the agent when you know exactly what you need:

```bash
flipside query create "SELECT * FROM ethereum.core.fact_blocks LIMIT 5"
```

### Interactive Chat

Start a conversation:

```bash
flipside chat              # Start interactive REPL
flipside chat repl         # Same thing
```

Or manage sessions programmatically:

```bash
flipside chat create                           # Create a session
flipside chat send-message "Hello" --session-id abc123
flipside chat list                             # List your sessions
flipside chat resume --session-id abc123       # Continue a conversation
```

---

## Agent Configuration

Agents are defined in YAML. Here's a real-world example:

### Chat Agent

```yaml
name: monad_validator
kind: chat
description: Monad validator monitoring and analytics agent

systemPrompt: |
  You are an expert Monad validator analyst helping operators monitor performance,
  analyze profitability, and make data-driven operational decisions.

  ## Your Capabilities

  **Historical & Current Analysis** (use SQL via monad.gov schema):
  - Validator identity, current state, and performance metrics
  - Historical earnings, rewards, and block production

  **Forward-Looking Projections** (use monad_pnl_projection_api skill):
  - Call use_skill("flipside/monad_pnl_projection_api") for any forward-looking analysis

  ## CRITICAL: SQL Query Rules

  **USE SKILLS FIRST.** Skills contain validated SQL query patterns.
  Call `use_skill()` to get the right queries rather than guessing.

skills:
  - flipside/monad_data           # SQL patterns for validator data
  - flipside/monad_sql_analytics  # Analytics queries
  - flipside/monad_decisions      # Decision support
  - snowflake                     # Global SQL skill

subAgents:
  - flipside/chart_publisher      # Delegate visualization to sub-agent

tools:
  - name: fetch_web_page
  - name: find_tables
  - name: get_table_schema
  - name: run_sql_query
  - name: use_skill
  - name: publish_html

metadata:
  model: claude-opus-4-5

maxTurns: 50
version: 1.0.0
status: active
visibility: organization
```

### Sub Agent (Structured I/O)

```yaml
name: data_formatter
kind: sub
description: Transform raw data into formatted output

systemPrompt: |
  You are a data formatting agent. Given input data, transform it according
  to the requested format and return structured output.

inputSchema:
  type: object
  properties:
    data:
      type: string
      description: The raw data to format
    format:
      type: string
      description: Output format (json, csv, markdown)
      enum: [json, csv, markdown]
  required:
    - data
    - format

outputSchema:
  type: object
  properties:
    formatted:
      type: string
      description: The formatted output
    row_count:
      type: integer
      description: Number of rows processed

metadata:
  model: claude-4-5-haiku

version: 1.0.0
status: draft
visibility: organization
```

### Key Fields

| Field | Description |
|-------|-------------|
| `name` | Lowercase identifier |
| `kind` | `chat` or `sub` |
| `systemPrompt` | Instructions for the LLM (camelCase) |
| `skills` | List of skill references (`org/skill_slug` or global name) |
| `subAgents` | Other agents this one can delegate to |
| `tools` | List of tools with `name:` syntax |
| `maxTurns` | Max conversation turns (camelCase) |
| `visibility` | `private`, `organization`, or `public` |
| `inputSchema` | JSON schema for sub agent inputs |
| `outputSchema` | JSON schema for sub agent outputs |

---

## Skills

Skills are the secret sauce. They bundle **tools + domain knowledge** that agents can invoke at runtime with `use_skill()`.

### Two Types of Skills

| Type | Reference | Example |
|------|-----------|---------|
| **Global** | Just the name | `snowflake` |
| **Org-scoped** | `org/skill_slug` | `flipside/monad_data` |

### Using Skills in Agents

```yaml
skills:
  - snowflake                     # Global skill
  - flipside/monad_data           # Org-scoped skill
  - flipside/monad_sql_analytics  # Another org skill
```

In your agent's system prompt, call skills at runtime:

```
Call use_skill("flipside/monad_pnl_projection_api") for any forward-looking analysis.
```

### Creating Skills

```bash
flipside skills init my_skill
```

Skills have a `knowledge` field where you put extensive domain documentation:

```yaml
name: Monad Validator Data
slug: monad_data
description: Query patterns and data access for Monad validator staking analytics
visibility: organization

tools:
  - find_tables
  - get_table_schema
  - run_sql_query
  - run_sql_for_download

knowledge: |
  # Monad Validator Data Access Patterns

  ## MANDATORY: Call get_table_schema BEFORE Any SQL Query

  **NEVER write SQL without first calling:**
  ```
  get_table_schema("monad.gov.TABLE_NAME")
  ```

  **Why?** Column names are NOT guessable:
  - dim_validators uses `validator_name`, NOT `label` or `name`
  - Tables have specific column names that must be verified

  ## Schema Overview

  | Table | Purpose |
  |-------|---------|
  | dim_validators | Validator dimension with name, addresses, commission |
  | ez_validator_earnings | Daily validator commission earnings |
  | ez_validator_apr | Daily and rolling APR/APY per validator |

  ## Example Query

  ```sql
  SELECT
      earning_date,
      validator_id,
      validator_name,
      total_earned,
      total_earned_usd
  FROM monad.gov.ez_validator_earnings
  ORDER BY earning_date DESC;
  ```
```

The `knowledge` field is where the real value lives. Put:
- SQL query patterns with examples
- Domain-specific rules and constraints
- API endpoints and response formats
- Best practices and common pitfalls

### Skill Commands

```bash
flipside skills init my_skill      # Create skill YAML
flipside skills push my_skill.skill.yaml
flipside skills list
flipside skills describe org/my_skill
flipside skills pull org/my_skill  # Download existing skill
flipside skills delete org/my_skill
```

---

## Sub-Agents

Agents can delegate work to other agents using the `subAgents` field. This is useful for:

- **Visualization**: Delegate chart generation to avoid token limits
- **Specialized tasks**: Let a focused agent handle specific operations
- **Structured outputs**: Sub-agents can have strict input/output schemas

### Example: Chart Publisher

```yaml
# In your main agent
subAgents:
  - flipside/chart_publisher

systemPrompt: |
  For any chart or visualization, use the chart_publisher sub-agent.
  DO NOT try to generate HTML directly - it will hit token limits.

  How to use chart_publisher:
  1. Prepare your data as structured JSON
  2. Call the sub-agent with: chartType, title, data.series, labels
  3. The sub-agent returns a URL to the published chart
```

Sub-agents receive structured input and return structured output, making them predictable building blocks.

---

## Available Tools

Tools are the primitives that skills and agents use. Common tools include:

| Tool | Description |
|------|-------------|
| `find_tables` | Semantic search to discover relevant tables |
| `get_table_schema` | Get column details for a specific table |
| `run_sql_query` | Execute SQL against the data warehouse |
| `run_sql_for_download` | Run SQL and get downloadable results |
| `query_results_with_duckdb` | Query previous results with DuckDB |
| `fetch_web_page` | Fetch content from a URL (for APIs) |
| `use_skill` | Invoke another skill at runtime |
| `publish_html` | Publish HTML visualizations to a public URL |
| `search_web` | Search the web for context |

### Listing Tools

```bash
flipside tools list              # List all available tools
flipside tools schema <tool>     # Get the schema for a specific tool
```

---

## Workflows

Automated DAG-based pipelines for recurring analysis.

```bash
# Create a workflow
flipside workflows init my_workflow

# Deploy
flipside workflows push my_workflow.yaml

# Run (with optional inputs)
flipside workflows run <id>
flipside workflows run <id> -i '{"chain": "ethereum"}'

# Check status
flipside workflows runs list <id>
```

Workflows can:
- Execute SQL queries
- Transform data with AI steps
- Send email notifications
- Reference uploaded CSV files

---

## Uploads

Upload CSV files to reference in SQL queries.

```bash
flipside uploads upload data.csv
flipside uploads upload data.csv --description "Q4 sales data"
flipside uploads list
flipside uploads get data.csv
flipside uploads delete data.csv
```

Reference in SQL with `${upload:filename}` syntax.

---

## Query Management

For saved queries and runs:

```bash
# Create and execute
flipside query create "SELECT COUNT(*) FROM ethereum.core.fact_blocks"
flipside query execute <query-id>

# Manage queries
flipside query list
flipside query get <query-id>
flipside query update <query-id> --name "New name"
flipside query delete <query-id>

# Check run status
flipside query-run status <run-id>
flipside query-run result <run-id>
flipside query-run data <run-id>
flipside query-run download <run-id>
flipside query-run poll <run-id> --timeout 5m
flipside query-run cancel <run-id>
```

---

## Global Flags

These work with any command:

| Flag | Description |
|------|-------------|
| `-j, --json` | Output as JSON (for scripting) |
| `-v, --verbose` | Show request/response details |
| `--endpoint` | Override API endpoint |
| `-e, --env` | Environment: `local`, `staging`, `prod` |

---

## Configuration

```bash
flipside config show          # View current config
flipside config set key value # Update a setting
flipside whoami               # Show current org and API config
```

---

## Staying Updated

```bash
flipside update
```

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "Not authenticated" | Run `flipside login` |
| "Agent not found" | Check `flipside agents list` — use `org/name` format |
| Validation errors | Run `flipside agents validate <file>` |
| Debug issues | Add `-v` for verbose output |

---

## Command Reference

```
flipside login                 Authenticate
flipside llm onboard           Guidance for AI agents
flipside quickstart [path]     Generate example agents

flipside agents list           List agents
flipside agents init <name>    Create agent YAML
flipside agents validate       Validate YAML
flipside agents push           Deploy agent
flipside agents run            Execute agent
flipside agents describe       View agent details
flipside agents pull           Download agent YAML
flipside agents delete         Delete agent

flipside skills list           List skills
flipside skills init           Create skill YAML
flipside skills push           Deploy skill
flipside skills describe       View skill details
flipside skills pull           Download skill YAML
flipside skills delete         Delete skill

flipside chat                  Interactive REPL
flipside chat create           Create session
flipside chat send-message     Send message
flipside chat list             List sessions
flipside chat resume           Resume session

flipside query create          Create saved query
flipside query execute         Run a query
flipside query list            List queries
flipside query-run status      Check run status
flipside query-run download    Download results

flipside workflows init        Create workflow
flipside workflows push        Deploy workflow
flipside workflows run         Trigger workflow
flipside workflows list        List workflows

flipside uploads upload        Upload CSV
flipside uploads list          List uploads

flipside config show           View config
flipside whoami                Current org info
flipside update                Update CLI
```

---

## Links

- [API Reference](./docs/API.md)
- [Flipside Docs](https://docs.flipsidecrypto.xyz)
