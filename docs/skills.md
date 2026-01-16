# Skills

Skills bundle **domain knowledge + tools** that agents invoke at runtime with `use_skill()`. They're how you teach agents about specific data domains.

## Skill Types

| Type | Reference | Example |
|------|-----------|---------|
| **Global** | Just the name | `snowflake` |
| **Org-scoped** | `org/skill_slug` | `flipside/monad_data` |

## Commands

```bash
flipside skills init my_skill        # Create skill YAML
flipside skills push <file>          # Deploy
flipside skills list                 # List skills
flipside skills describe org/skill   # View details
flipside skills pull org/skill       # Download existing skill
flipside skills delete org/skill     # Delete
```

## YAML Structure

```yaml
name: Monad Validator Data
slug: monad_data
description: Query patterns for Monad validator analytics
visibility: organization

tools:
  - find_tables
  - get_table_schema
  - run_sql_query

knowledge: |
  # How to Query Monad Validator Data

  ## Always call get_table_schema first
  Column names aren't guessable. Verify before writing SQL.

  ## Key Tables
  | Table | Purpose |
  |-------|---------|
  | dim_validators | Validator names, addresses, commission |
  | ez_validator_earnings | Daily commission earnings |
  | ez_validator_apr | Daily and rolling APR/APY |

  ## Example Query
  ```sql
  SELECT validator_name, total_earned_usd
  FROM monad.gov.ez_validator_earnings
  ORDER BY earning_date DESC
  ```
```

## Fields Reference

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Human-readable name |
| `slug` | Yes | URL-safe identifier |
| `description` | Yes | What the skill provides |
| `visibility` | No | `private`, `organization`, or `public` |
| `tools` | No | Tools this skill provides access to |
| `knowledge` | Yes | Domain documentation for the LLM |

## The Knowledge Field

This is where the value lives. Include:

- **SQL patterns** with working examples
- **Schema documentation** (tables, columns, relationships)
- **Domain rules** and constraints
- **Common pitfalls** and how to avoid them
- **API formats** if the skill wraps external services

Example knowledge section:

```markdown
# Ethereum DEX Analytics

## Required: Always verify schema first
Call `get_table_schema("ethereum.defi.ez_dex_swaps")` before writing queries.

## Key Tables

| Table | Use For |
|-------|---------|
| ez_dex_swaps | Individual swap transactions |
| ez_dex_pools | Pool metadata and TVL |

## Common Queries

### Top pools by volume (24h)
```sql
SELECT pool_name, SUM(amount_usd) as volume
FROM ethereum.defi.ez_dex_swaps
WHERE block_timestamp > CURRENT_DATE - 1
GROUP BY pool_name
ORDER BY volume DESC
LIMIT 10
```

## Gotchas
- `amount_usd` can be NULL for unknown tokens
- Always filter by `block_timestamp` to avoid scanning full history
```

## Using Skills in Agents

Reference skills in your agent YAML:

```yaml
skills:
  - snowflake                 # Global skill
  - flipside/monad_data       # Org-scoped skill
```

Agents call skills at runtime:

```
use_skill("flipside/monad_data")
```

The skill's knowledge and tools become available to the agent for that turn.
