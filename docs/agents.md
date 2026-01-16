# Agents

Agents are LLM-powered assistants that can query data, use tools, and invoke skills.

## Agent Types

| Type | Flag | Use Case |
|------|------|----------|
| **Chat** | `--message "..."` | Natural language conversations |
| **Sub** | `--data-json '{...}'` | Structured input/output for pipelines |

## Commands

```bash
flipside agents init my_agent             # Create agent YAML
flipside agents init my_agent --kind sub  # Create sub-agent
flipside agents validate <file>           # Validate before deploy
flipside agents push <file>               # Deploy
flipside agents run org/name --message "..." # Run chat agent
flipside agents run org/name --data-json '{...}' # Run sub-agent
flipside agents describe org/name         # View details
flipside agents delete org/name           # Delete
```

## YAML Structure

### Chat Agent

```yaml
name: my_agent
kind: chat
description: What this agent does

systemPrompt: |
  Your instructions to the LLM.
  Tell it what tools to use and when.

skills:
  - snowflake                 # Global skill
  - flipside/monad_data       # Org-scoped skill

subAgents:
  - flipside/chart_publisher  # Delegate tasks to other agents

tools:
  - name: run_sql_query
  - name: use_skill
  - name: publish_html

maxTurns: 50
visibility: organization      # private | organization | public
```

### Sub-Agent (Structured I/O)

Sub-agents receive structured input and return structured output. They're useful for pipelines and delegation.

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
      enum: [json, csv, markdown]
  required: [data, format]

outputSchema:
  type: object
  properties:
    formatted:
      type: string
    row_count:
      type: integer

metadata:
  model: claude-4-5-haiku

version: 1.0.0
visibility: organization
```

## Fields Reference

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Lowercase identifier |
| `kind` | Yes | `chat` or `sub` |
| `description` | Yes | What the agent does |
| `systemPrompt` | Yes | Instructions for the LLM |
| `skills` | No | Skills the agent can invoke |
| `subAgents` | No | Other agents to delegate to |
| `tools` | No | Tools available to the agent |
| `maxTurns` | No | Max conversation turns (default: 50) |
| `visibility` | No | `private`, `organization`, or `public` |
| `inputSchema` | Sub only | JSON schema for input |
| `outputSchema` | Sub only | JSON schema for output |
| `metadata.model` | No | Model to use (e.g., `claude-opus-4-5`) |

## Using Sub-Agents

Agents can delegate work using the `subAgents` field:

```yaml
subAgents:
  - flipside/chart_publisher

systemPrompt: |
  For visualizations, delegate to chart_publisher with structured data.
  Don't generate HTML directly—it hits token limits.
```

Sub-agents are useful for:
- **Visualization**: Delegate chart generation
- **Specialized tasks**: Let focused agents handle specific operations
- **Token management**: Offload large outputs to dedicated agents
