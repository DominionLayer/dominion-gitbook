# Logging & Reports

Dominion provides comprehensive logging and reporting for auditability and debugging.

## Logging

### Log Format

All logs are structured JSON:

```json
{
  "timestamp": "2026-01-04T12:00:00.000Z",
  "level": "info",
  "message": "Analysis completed",
  "context": {
    "run_id": "run_abc123",
    "agent_id": "analyst_042",
    "market_id": "eth_jan_2026",
    "duration_ms": 1250
  }
}
```

### Log Levels

| Level | Use Case |
|-------|----------|
| `debug` | Detailed debugging info |
| `info` | Normal operations |
| `warn` | Potential issues |
| `error` | Errors that need attention |

### Configuration

```yaml
# pm.config.yaml
logging:
  level: "info"
  format: "json"
  file: "./logs/polymarket.log"
```

### Environment Override

```bash
export LOG_LEVEL=debug
```

## Run IDs

Every execution session has a unique `run_id`:

```
run_2026-01-04T12-00-00-abc123
```

Use run IDs to:
- Correlate logs across components
- Reproduce analyses
- Generate reports

### Finding Run IDs

```bash
# List recent runs
dominion-pm report --list

# Output:
# run_2026-01-04T12-00-00-abc123  analyze  completed
# run_2026-01-04T11-30-00-def456  scan     completed
# run_2026-01-04T11-00-00-ghi789  compare  failed
```

## Reports

### Report Types

| Type | Format | Content |
|------|--------|---------|
| Analysis | JSON + MD | Probability estimates, factors, confidence |
| Comparison | JSON + MD | Market rankings, edge calculations |
| Simulation | JSON + MD | EV calculations, scenarios |
| Audit | JSON | Full execution trace |

### Generating Reports

```bash
# Generate during analysis
dominion-pm analyze <market_id> --report

# Regenerate from run ID
dominion-pm report run_abc123

# Generate for latest run
dominion-pm report latest
```

### Report Location

```
reports/
├── analysis-2026-01-04T12-00-00.json
├── analysis-2026-01-04T12-00-00.md
├── compare-2026-01-04T11-30-00.json
├── compare-2026-01-04T11-30-00.md
└── ...
```

### Report Structure (JSON)

```json
{
  "meta": {
    "run_id": "run_abc123",
    "type": "analysis",
    "timestamp": "2026-01-04T12:00:00.000Z",
    "version": "1.0.0"
  },
  "input": {
    "market_id": "eth_jan_2026",
    "estimator": "llm"
  },
  "output": {
    "market": {
      "question": "Will ETH be above $4000 on Jan 31?",
      "current_price": 0.62,
      "volume_24h": 150000
    },
    "analysis": {
      "estimated_probability": 0.68,
      "confidence": 0.72,
      "edge": 0.06,
      "key_factors": ["..."],
      "assumptions": ["..."],
      "failure_modes": ["..."]
    }
  },
  "disclaimer": "This is analysis only, not financial advice."
}
```

### Report Structure (Markdown)

```markdown
# Market Analysis Report

**Run ID:** run_abc123  
**Date:** 2026-01-04T12:00:00Z  

## Market

**Question:** Will ETH be above $4000 on Jan 31?  
**Current Price:** $0.62 (62% implied probability)  
**24h Volume:** $150,000  

## Analysis

| Metric | Value |
|--------|-------|
| Model Probability | 68% |
| Confidence | 72% |
| Edge | +6% |

### Key Factors
- Factor 1
- Factor 2

### Assumptions
- Assumption 1

### Failure Modes
- Could fail if...

---

*Disclaimer: This is analysis and simulation, not financial advice.*
```

## Database Logging

All operations are logged to SQLite:

### Tables

```sql
-- Runs
CREATE TABLE runs (
  id TEXT PRIMARY KEY,
  type TEXT,
  status TEXT,
  started_at INTEGER,
  completed_at INTEGER,
  config TEXT
);

-- Analyses
CREATE TABLE analyses (
  id TEXT PRIMARY KEY,
  run_id TEXT,
  market_id TEXT,
  estimator TEXT,
  result TEXT,
  created_at INTEGER
);

-- Simulations
CREATE TABLE simulations (
  id TEXT PRIMARY KEY,
  run_id TEXT,
  market_id TEXT,
  params TEXT,
  result TEXT,
  created_at INTEGER
);
```

### Querying History

```bash
# Via CLI
dominion-pm history --market <id>
dominion-pm history --days 7

# Via SQLite directly
sqlite3 data/polymarket.db "SELECT * FROM analyses ORDER BY created_at DESC LIMIT 10"
```

## Audit Trail

For compliance and debugging, every action has an audit trail:

```json
{
  "audit_id": "audit_xyz",
  "run_id": "run_abc123",
  "action": "llm_complete",
  "timestamp": "2026-01-04T12:00:00.000Z",
  "input": {
    "provider": "openai",
    "model": "gpt-4o-mini",
    "messages": "[REDACTED]",
    "message_count": 2
  },
  "output": {
    "tokens_used": 1250,
    "latency_ms": 850,
    "status": "success"
  }
}
```

### Privacy

By default, prompt contents are **not logged**. Only metadata is stored:
- Message count
- Token counts
- Model used
- Latency

To enable full logging (dev only):
```yaml
logging:
  include_prompts: true  # NOT recommended for production
```

## Best Practices

1. **Always use run IDs** — Include in bug reports
2. **Set appropriate log level** — `info` for prod, `debug` for dev
3. **Rotate logs** — Use logrotate or similar
4. **Archive reports** — Keep for compliance
5. **Monitor errors** — Alert on error rate spikes

