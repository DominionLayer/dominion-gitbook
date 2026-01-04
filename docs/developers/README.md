# Developer Documentation

Everything you need to build with Dominion.

## Quick Links

- [Quickstart](quickstart.md) — Get running in 5 minutes
- [Configuration](configuration.md) — Config files and environment
- [Plugin System](plugins.md) — Extending the CLI
- [LLM Providers](llm-providers.md) — Using the Gateway
- [Logging & Reports](logging.md) — Debugging and audit trails
- [Testing](testing.md) — Writing and running tests

## Prerequisites

- Node.js 20+
- pnpm (recommended) or npm
- A Dominion API token

## Installation

```bash
# Install CLI tools
pnpm add -g @dominion/cli @dominion/polymarket-cli

# Configure
export DOMINION_API_TOKEN=dom_your_token_here

# Verify
dominion-pm whoami
```

## Architecture Overview

```
Your Application
      │
      ▼
┌─────────────────┐
│   CLI Tools     │  ← dominion, dominion-pm
└────────┬────────┘
         │
┌────────▼────────┐
│    Gateway      │  ← LLM access, auth
└────────┬────────┘
         │
┌────────▼────────┐
│  LLM Providers  │  ← OpenAI, Anthropic
└─────────────────┘
```

## Getting Help

- **Docs:** You're here!
- **GitHub:** [github.com/DominionLayer](https://github.com/DominionLayer)
- **Discord:** [discord.gg/dominion](https://discord.gg/dominion)

