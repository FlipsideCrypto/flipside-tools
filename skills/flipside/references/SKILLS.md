# Creating Flipside Skills

Skills bundle tools and domain knowledge for agents. They're reusable packages that give agents specific capabilities.

> **Note:** These are Flipside platform skills (used by Flipside agents), not Agent Skills (the package format this skill uses).

## Skill Types

| Type | Reference Format | Example |
|------|------------------|---------|
| Global | Just the name | `snowflake` |
| Org-scoped | `org/skill_slug` | `flipside/defi_analysis` |

## Using Skills in Agents

Reference skills in your agent's YAML:

```yaml
name: my_agent
kind: chat
skills:
  - snowflake                     # Global skill
  - flipside/defi_analysis        # Org-scoped skill
systemPrompt: |
  You are a DeFi analyst...
```

## Creating a Skill

### 1. Initialize

```bash
flipside skills init my_skill
# Creates: my_skill.skill.yaml
```

### 2. Define the Skill

```yaml
name: My Custom Skill
slug: my_skill
description: What this skill provides

# Tools this skill grants access to
tools:
  - run_sql_query
  - find_tables
  - get_table_schema

# Domain knowledge
knowledge: |
  # DeFi Analysis Guidelines

  ## Key Tables
  - ethereum.defi.ez_dex_swaps - DEX swap transactions
  - ethereum.defi.ez_lending_borrows - Lending protocol borrows

  ## Best Practices
  - Always filter by date to limit result size
  - Use LOWER() for address comparisons
  - Check token decimals before calculating amounts

  ## Example Queries

  ### Top DEX Swaps
  ```sql
  SELECT
    platform,
    SUM(amount_in_usd) as volume
  FROM ethereum.defi.ez_dex_swaps
  WHERE block_timestamp >= CURRENT_DATE - 7
  GROUP BY platform
  ORDER BY volume DESC
  ```
```

### 3. Deploy

```bash
flipside skills push my_skill.skill.yaml
# Output: Deployed as username/my_skill
```

## Skill YAML Schema

```yaml
# Required
name: Human Readable Name        # Display name
slug: machine_name               # URL-safe identifier (lowercase, underscores)

# Optional
description: What this skill does and when to use it

# Tools to include
tools:
  - run_sql_query               # Execute SQL
  - find_tables                 # Search for tables
  - get_table_schema            # Get column definitions
  - create_visualization        # Create charts
  - web_search                  # Search the web

# Domain knowledge (markdown)
knowledge: |
  # Instructions and examples

  This knowledge is injected into the agent's context
  when the skill is activated.

  Include:
  - Relevant table references
  - SQL patterns and examples
  - Domain-specific best practices
  - Common pitfalls to avoid
```

## Available Tools

| Tool | Description |
|------|-------------|
| `run_sql_query` | Execute SQL against Flipside data warehouse |
| `find_tables` | Search for tables by keyword |
| `get_table_schema` | Get column definitions for a table |
| `create_visualization` | Generate charts from data |
| `web_search` | Search the internet |

## Managing Skills

```bash
# List skills
flipside skills list

# Push/update skill
flipside skills push my_skill.skill.yaml

# The output shows the full reference to use in agents
# e.g., "Deployed as username/my_skill"
```

## Best Practices

1. **Focus on a domain** - One skill per topic (DeFi, NFTs, governance)
2. **Include examples** - SQL snippets help agents understand patterns
3. **Document tables** - List the key tables and their purposes
4. **Add gotchas** - Common mistakes and how to avoid them
5. **Keep knowledge concise** - Too much text dilutes the signal

## Example: DeFi Analysis Skill

```yaml
name: DeFi Analysis
slug: defi_analysis
description: Tools and knowledge for analyzing DeFi protocols

tools:
  - run_sql_query
  - find_tables
  - get_table_schema

knowledge: |
  # DeFi Analysis on Flipside

  ## Core Tables

  | Table | Description |
  |-------|-------------|
  | ethereum.defi.ez_dex_swaps | DEX swap transactions |
  | ethereum.defi.ez_lending_borrows | Lending borrows |
  | ethereum.defi.ez_lending_repayments | Lending repayments |
  | ethereum.defi.ez_bridge_activity | Cross-chain bridges |

  ## Key Patterns

  ### Protocol Volume
  ```sql
  SELECT
    platform,
    COUNT(*) as tx_count,
    SUM(amount_in_usd) as volume_usd
  FROM ethereum.defi.ez_dex_swaps
  WHERE block_timestamp >= CURRENT_DATE - 7
  GROUP BY platform
  ORDER BY volume_usd DESC
  ```

  ### Whale Tracking
  ```sql
  SELECT *
  FROM ethereum.defi.ez_dex_swaps
  WHERE amount_in_usd > 100000
    AND block_timestamp >= CURRENT_DATE - 1
  ORDER BY amount_in_usd DESC
  ```

  ## Tips
  - USD values are pre-calculated in ez_ tables
  - Always filter by date for performance
  - Use platform column to filter by protocol
```
