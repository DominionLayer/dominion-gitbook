# Polymarket CLI

The Polymarket Prediction CLI (`dominion-pm`) is a tool for analyzing prediction markets. It is **not for automated trading**.

> **Disclaimer:** This tool provides analysis and simulation only. It does NOT provide financial advice and does NOT execute trades automatically.

## Overview

The CLI helps users:

* Discover active Polymarket markets
* Estimate outcome probabilities
* Calculate edge (model vs market)
* Simulate positions and expected value
* Generate analysis reports



## Installation

```bash
# Clone the repository
git clone https://github.com/DominionLayer/prediction-layer
cd prediction-layer

# Install dependencies
pnpm install

# Build
pnpm build

# Link for global usage
pnpm link --global
```

## Quick Start

```bash
# Initialize configuration and database
dominion-pm init

# Login with your API token
dominion-pm login
# Enter your dom_xxx token when prompted

# Scan active markets
dominion-pm scan

# Analyze a specific market
dominion-pm analyze <market_id>

# Simulate a position
dominion-pm simulate <market_id> --position YES --size 10
```

## Commands

| Command                       | Description                    |
| ----------------------------- | ------------------------------ |
| `dominion-pm init`            | Initialize config and database |
| `dominion-pm login`           | Authenticate with gateway      |
| `dominion-pm whoami`          | Show user and quota            |
| `dominion-pm scan`            | Fetch active markets           |
| `dominion-pm show <id>`       | View market details            |
| `dominion-pm analyze <id>`    | Run probability estimation     |
| `dominion-pm compare`         | Find top edge opportunities    |
| `dominion-pm simulate <id>`   | Calculate expected value       |
| `dominion-pm report`          | Generate reports               |
| `dominion-pm doctor`          | Validate setup                 |
| `dominion-pm help-disclaimer` | View legal notices             |

## Probability Estimation

### LLM Estimator

Uses AI models to analyze market context:

```bash
dominion-pm analyze <market_id>
```

Output:

```json
{
  "estimated_probability": 0.65,
  "confidence": 0.72,
  "key_factors": [
    "Recent polling data",
    "Historical patterns"
  ],
  "assumptions": [
    "No major events before resolution"
  ],
  "failure_modes": [
    "Unexpected announcement",
    "Data quality issues"
  ]
}
```

### Baseline Estimator

Deterministic heuristics (no LLM required):

```bash
dominion-pm analyze <market_id> --estimator baseline
```

Uses:

* Price momentum
* Liquidity levels
* Time to expiry
* Bid-ask spread

## Edge Calculation

```
edge = model_probability - market_probability
```

| Edge  | Interpretation                  |
| ----- | ------------------------------- |
| > +5% | Model thinks YES is undervalued |
| < -5% | Model thinks NO is undervalued  |
| ±5%   | Within noise range              |

```bash
# Find markets with highest edge
dominion-pm compare --min-edge 0.05 --top 10
```

## Simulation

Simulate positions **without executing**:

```bash
dominion-pm simulate <market_id> \
  --position YES \
  --size 10 \
  --entry-price 0.60
```

Output:

```
Position Simulation
-------------------
Market: Will X happen?
Position: YES
Size: 10 contracts
Entry Price: $0.60

Scenarios (Model Prob: 65%):
  Optimistic (75%): +$1.50 EV
  Base (65%):       +$0.50 EV  
  Pessimistic (55%): -$0.50 EV

Break-even: 60%
Kelly Fraction: 12.5%

[This is simulation only. Not financial advice.]
```

## Authentication

### Getting Your API Token

Contact the Dominion administrator to receive a `dom_xxx` API token.

### Login

```bash
dominion-pm login
```

You will be prompted for:

* **Gateway URL**: Press Enter to use the default (`https://api.dominionlayer.io`)
* **API Token**: Enter your `dom_xxx` token

The token is verified and saved to `pm.config.yaml` for future use.

### Alternative: Environment Variable

```bash
export DOMINION_API_TOKEN=dom_your_token
```

## Configuration

### pm.config.yaml

```yaml
# Gateway
gateway:
  url: "https://api.dominionlayer.io"
  token: "dom_your_token"

# Polymarket API
polymarket:
  base_url: "https://clob.polymarket.com"
  gamma_url: "https://gamma-api.polymarket.com"

# Scoring filters
scoring:
  min_liquidity: 1000
  min_volume: 100
  max_spread: 0.10

# Simulation parameters
simulation:
  fee_bps: 100
  slippage_bps: 50
  confidence_band: 0.10
```

## Reports

Generate JSON and Markdown reports:

```bash
# Analyze and save report
dominion-pm analyze <market_id> --report

# Reports saved to ./reports/
ls reports/
# analysis-2026-01-04.json
# analysis-2026-01-04.md
```

## Safety Features

### Every Output Includes

* "This is analysis and simulation, not financial advice"
* Confidence/uncertainty ratings
* Assumption disclosures
* Failure mode warnings

### No Automatic Execution

The `exec` command generates intent JSON but does **not** execute:

```bash
dominion-pm exec <market_id> --position YES --size 10

# Outputs intent JSON only
# Requires --approve flag to generate
# Still does NOT execute trades
```

### Rate Limiting

Built-in rate limiting protects against:

* API abuse
* Excessive LLM costs
* Accidental loops

## Example Workflow

```bash
# 1. Setup
dominion-pm init
export DOMINION_API_TOKEN=dom_xxx

# 2. Explore markets
dominion-pm scan --query "election"

# 3. Deep dive on interesting market
dominion-pm show abc123
dominion-pm analyze abc123

# 4. Find best opportunities
dominion-pm compare --min-liquidity 10000 --top 20

# 5. Simulate a position
dominion-pm simulate abc123 --position YES --size 5

# 6. Generate report
dominion-pm analyze abc123 --report

# 7. Make your own decision (this tool doesn't trade for you)
```

## Legal Disclaimer

**IMPORTANT:** Run `dominion-pm help-disclaimer` for full legal notices.

Summary:

* This software is for informational and educational purposes ONLY
* It does NOT provide financial, investment, or trading advice
* It does NOT guarantee any profits or outcomes
* It does NOT execute trades automatically
* Past performance does not indicate future results
* Use at your own risk

***

_Dominion is experimental software. Prediction markets involve financial risk. Never risk more than you can afford to lose._
