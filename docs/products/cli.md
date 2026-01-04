# Dominion CLI

The Dominion CLI is a command-line tool for running and managing autonomous agent swarms.

## Overview

The CLI provides:
- Agent orchestration and management
- Plugin-based extensibility
- Workflow automation
- Safety gates and approvals
- Comprehensive logging

## Installation

```bash
# Clone the repository
git clone https://github.com/DominionLayer/dominion-swarm
cd dominion-swarm

# Install dependencies
npm install

# Build
npm run build

# Link for global usage
npm link
```

## Quick Start

```bash
# Initialize configuration
dominion init

# Check setup
dominion doctor

# Run an analysis workflow
dominion run analyze --target "ETH price prediction"

# View agent status
dominion agents list
```

## Core Commands

| Command | Description |
|---------|-------------|
| `dominion init` | Initialize configuration |
| `dominion doctor` | Validate setup |
| `dominion agents` | Manage agents |
| `dominion tasks` | Manage tasks |
| `dominion run` | Execute workflows |
| `dominion queue` | View job queue |
| `dominion gov` | Governance commands |

## Plugin System

Dominion uses a modular plugin architecture:

### Built-in Plugins

| Plugin | Description |
|--------|-------------|
| `observe` | Watch data sources |
| `analyze` | LLM-based analysis |
| `execute` | Action execution |
| `infra` | Job scheduling |
| `market` | Market simulation |
| `governance` | Proposal management |
| `selfimprove` | Performance tracking |

### Plugin Structure

```typescript
export class MyPlugin extends Plugin {
  readonly name = 'my-plugin';
  readonly version = '1.0.0';
  
  async initialize(): Promise<void> {
    // Setup code
  }
  
  getTools(): Tool[] {
    return [
      // Tool definitions
    ];
  }
}
```

See [Plugin System](../developers/plugins.md) for development details.

## Configuration

### dominion.config.yaml

```yaml
# General settings
general:
  name: "My Dominion Instance"
  environment: "development"

# LLM Provider (via Gateway)
provider:
  default: "gateway"

# Gateway configuration
gateway:
  url: "https://web-production-2fb66.up.railway.app"
  # Token set via DOMINION_API_TOKEN env var

# Safety settings
safety:
  dry_run_default: true
  require_approval: true
  max_concurrent_tasks: 10

# Logging
logging:
  level: "info"
  format: "json"
  file: "./logs/dominion.log"
```

### Environment Variables

| Variable | Description |
|----------|-------------|
| `DOMINION_API_TOKEN` | Gateway authentication |
| `LOG_LEVEL` | Logging verbosity |
| `DATABASE_PATH` | SQLite database path |

## Workflows

Workflows define multi-step agent operations:

```yaml
# workflows/analyze-market.yaml
name: analyze-market
description: Analyze a prediction market

steps:
  - name: observe
    plugin: observe
    tool: fetch_market_data
    params:
      market_id: "{{ inputs.market_id }}"
  
  - name: analyze
    plugin: analyze
    tool: llm_analysis
    params:
      data: "{{ steps.observe.output }}"
      prompt: "Analyze this market data..."
  
  - name: report
    plugin: execute
    tool: write_file
    params:
      path: "./reports/{{ inputs.market_id }}.md"
      content: "{{ steps.analyze.output }}"
```

## Safety Features

### Dry Run Default
All actions run in dry-run mode unless explicitly approved.

```bash
# Preview what would happen
dominion run workflow --dry-run

# Actually execute (requires approval)
dominion run workflow --approve
```

### Approval Gates
Actions pass through approval checks:
- Policy compliance
- Risk limits
- Rate limits
- Optional human review

### Audit Logging
Every action is logged with:
- Timestamp
- Agent ID
- Action details
- Approval chain
- Outcome

## Example: Market Analysis

```bash
# 1. Initialize
dominion init

# 2. Configure (edit dominion.config.yaml)

# 3. Run market analysis
dominion run analyze-market \
  --market-id "eth_jan_2026" \
  --output-format markdown

# 4. View results
cat reports/eth_jan_2026.md
```

## Troubleshooting

### Common Issues

**"DOMINION_API_TOKEN not set"**
```bash
export DOMINION_API_TOKEN=dom_your_token
```

**"Database not initialized"**
```bash
dominion init --force
```

**"Plugin not found"**
```bash
dominion plugins list
dominion plugins install <plugin-name>
```

## Further Reading

- [Configuration Reference](../developers/configuration.md)
- [Plugin Development](../developers/plugins.md)
- [Workflow Reference](../developers/workflows.md)

