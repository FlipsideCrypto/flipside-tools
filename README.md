# Flipside Tools

AI agent skills and CLI tools for querying blockchain data across 40+ chains.

## Install the Skill

```bash
npx skills add FlipsideCrypto/flipside-tools
```

Or add to your project manually by copying `skills/flipside/` to your `.claude/skills/` directory.

## Install the CLI

```bash
curl -fsSL https://raw.githubusercontent.com/FlipsideCrypto/flipside-tools/main/install.sh | sh
```

Then authenticate:

```bash
flipside login
```

## Quick Start

```bash
# Start an interactive chat with the SQL agent
flipside chat

# Run a one-off query through an agent
flipside agents run flipside/sql_agent --message "Show me the largest ETH transfers today"

# Discover tables
flipside tools find_tables "token transfers"
```

## What's in this repo

```
flipside-tools/
├── skills/
│   └── flipside/       # AI agent skill for blockchain analytics
│       ├── SKILL.md    # Main skill definition
│       ├── references/ # Detailed docs (agents, automations, queries)
│       ├── scripts/    # Validation helpers
│       └── assets/     # YAML templates
├── install.sh          # CLI installer
└── README.md
```

## Documentation

- [Flipside Docs](https://docs.flipsidecrypto.xyz)
- [CLI Reference](https://docs.flipsidecrypto.xyz/get-started/cli)
- [Skill Reference](skills/flipside/SKILL.md)

## License

MIT
