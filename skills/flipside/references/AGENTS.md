# Creating and Running Agents

Agents are AI-powered assistants that can query blockchain data, analyze results, and perform complex workflows.

## Agent Types

| Type | Use Case | Invocation |
|------|----------|------------|
| `chat` | Conversational, open-ended questions | `--message "..."` |
| `sub` | Structured I/O, programmatic use | `--data-json '{...}'` |

### Chat Agents

Best for exploratory analysis and natural language questions:

```bash
flipside agents run flipside/sql_agent --message "What are the top DEX protocols by volume today?"
```

### Sub Agents

Best for structured workflows and automation:

```bash
flipside agents run flipside/token_analyzer --data-json '{"chain": "ethereum", "token_address": "0x..."}'
```

Sub agents have defined input/output schemas for predictable behavior.

## Agent YAML Schema

```yaml
# Required fields
name: my_agent                    # Lowercase, underscores OK
kind: chat                        # chat or sub

# Visibility
visibility: personal              # personal, organization, or public

# Skills (optional - flipside/data is auto-injected)
# The flipside/data skill is automatically injected into ALL agents,
# providing core SQL query tools. You only need to add skills here for additional capabilities.
skills:
  - flipside/my_custom_skill      # Org-scoped skill

# Instructions
systemPrompt: |
  You are an expert blockchain data analyst.
  Use the available tools to answer questions about on-chain data.
  Always explain your methodology.

# Limits
maxTurns: 10                      # Max conversation turns

# Sub agents (for delegation)
subAgents:
  - flipside/token_analyzer
  - flipside/wallet_profiler

# For sub agents only
inputSchema:
  type: object
  properties:
    chain:
      type: string
      enum: [ethereum, solana, polygon]
    address:
      type: string
  required: [chain, address]

outputSchema:
  type: object
  properties:
    summary:
      type: string
    risk_score:
      type: number
```

## Creating an Agent

### 1. Initialize

```bash
flipside agents init my_agent
# Creates: my_agent.agent.yaml
```

### 2. Edit the YAML

Customize the generated template with your skills and prompt.

### 3. Validate

```bash
flipside agents validate my_agent.agent.yaml
```

### 4. Deploy

```bash
flipside agents push my_agent.agent.yaml
# Output: Deployed as username/my_agent
```

### 5. Run

```bash
flipside agents run username/my_agent --message "Test question"
```

## Skills in Agents

Skills bundle tools and domain knowledge. Reference them by name:

```yaml
skills:
  - flipside/defi_analysis        # Org-scoped skill
```

### Auto-Injected Skill

The `flipside/data` skill is **automatically injected** into all agents. This provides:
- `find_tables` - Search for tables by keyword (semantic search)
- `find_tables_by_name` - Search for tables by name pattern
- `get_table_schema` - Get column definitions for a table
- `run_query` - Execute a new SQL query
- `update_query` - Update an existing query's SQL
- `run_query_by_id` - Execute an existing query by ID
- `get_query_results` - Fetch more rows from a completed query

You don't need to add this skill manually—every agent gets SQL query capabilities by default.

### Adding Custom Skills

Add org-scoped skills for additional capabilities:

```bash
flipside skills list
```

## Sub-Agent Delegation

Chat agents can delegate to sub-agents for specialized tasks:

```yaml
name: research_agent
kind: chat
# flipside/data is auto-injected - SQL tools work out of the box

subAgents:
  - flipside/token_analyzer       # Structured token analysis
  - flipside/wallet_profiler      # Wallet behavior analysis

systemPrompt: |
  You are a research coordinator. When users ask about:
  - Token analysis: delegate to token_analyzer
  - Wallet behavior: delegate to wallet_profiler
  - General SQL: handle directly with your SQL tools
```

## Managing Agents

```bash
# List your agents
flipside agents list

# Get agent details
flipside agents describe username/my_agent

# Download agent config
flipside agents pull username/my_agent > my_agent.agent.yaml

# Update agent (edit YAML, then push)
flipside agents push my_agent.agent.yaml

# Delete agent
flipside agents delete username/my_agent
```

## Running with Options

```bash
# Add a title (helpful for tracking)
flipside agents run org/agent --message "..." --title "Testing new feature"

# JSON output
flipside agents run org/agent --message "..." --json

# Verbose mode (show API calls)
flipside agents run org/agent --message "..." --verbose
```

## Best Practices

1. **Use descriptive names** - `wallet_risk_analyzer` not `agent1`
2. **Write clear system prompts** - Explain the agent's role and capabilities
3. **SQL tools are automatic** - The `flipside/data` skill is auto-injected
4. **Use sub-agents for specialization** - Delegate complex subtasks
5. **Add --title when testing** - Makes runs easier to find in the UI
6. **Validate before pushing** - Catches YAML errors early
