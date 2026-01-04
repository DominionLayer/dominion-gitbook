# Configuration

How to configure Dominion CLIs.

## Config File

Create `pm.config.yaml`:

```yaml
general:
  name: "my-project"
  dry_run: true

gateway:
  url: "https://web-production-2fb66.up.railway.app"

provider:
  default: "openai"
  openai:
    model: "gpt-4o-mini"
    temperature: 0.2

polymarket:
  api_url: "https://api.polymarket.com"

scoring:
  min_volume: 1000
  min_liquidity: 500

simulation:
  default_scenarios: 3

reporting:
  output_dir: "./reports"
  formats: ["json", "markdown"]

logging:
  level: "info"
  format: "json"

database:
  path: "./data/polymarket.db"
```

## Environment Variables

```bash
export DOMINION_API_TOKEN=dom_xxx
export LOG_LEVEL=debug
```

## Loading Order

1. Default values
2. Config file
3. Environment variables
4. CLI flags (highest priority)
