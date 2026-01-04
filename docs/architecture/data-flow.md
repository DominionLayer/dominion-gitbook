# Data Flow

## Overview

Data flows through Dominion in a structured pipeline:

```
Observe → Analyze → Propose → Approve → Execute → Verify
```

Each stage has clear inputs, outputs, and responsibilities.

## Stage 1: Observe

### Input
- External data sources (APIs, chains, feeds)
- Trigger conditions

### Process
1. Watcher agents poll or subscribe to sources
2. Raw data is normalized to standard schemas
3. Observations are timestamped and signed
4. Data is pushed to observation queue

### Output
```json
{
  "type": "observation",
  "source": "coingecko_api",
  "timestamp": "2026-01-04T12:00:00Z",
  "agent_id": "watcher_001",
  "data": {
    "asset": "ETH",
    "price_usd": 3500.00,
    "volume_24h": 15000000000
  },
  "signature": "0x..."
}
```

## Stage 2: Analyze

### Input
- Observations from queue
- Historical data
- Market context

### Process
1. Analyst agents fetch relevant observations
2. LLM or model processes data
3. Analysis generates structured output
4. Confidence scores are assigned

### Output
```json
{
  "type": "analysis",
  "timestamp": "2026-01-04T12:00:05Z",
  "agent_id": "analyst_042",
  "market_id": "eth_price_jan",
  "result": {
    "estimated_probability": 0.65,
    "confidence": 0.72,
    "key_factors": [
      "Strong volume increase",
      "Positive funding rates"
    ],
    "assumptions": [
      "No major macro events"
    ],
    "failure_modes": [
      "Regulatory announcement",
      "Exchange outage"
    ]
  },
  "model": "gpt-4o",
  "tokens_used": 1250
}
```

## Stage 3: Propose

### Input
- Analysis results
- Market signals
- Current state

### Process
1. Coordinator evaluates analyses
2. Market prices inform decision
3. Proposal is generated
4. Risk assessment attached

### Output
```json
{
  "type": "proposal",
  "timestamp": "2026-01-04T12:00:10Z",
  "agent_id": "coordinator_007",
  "action": {
    "type": "market_trade",
    "market_id": "eth_price_jan",
    "direction": "YES",
    "amount": 100,
    "max_price": 0.68
  },
  "rationale": "Swarm consensus at 65%, edge of 5%",
  "risk_score": 0.3,
  "requires_approval": ["policy", "risk"]
}
```

## Stage 4: Approve

### Input
- Proposal
- Policy rules
- Risk limits

### Process
1. Policy check: Does this comply with rules?
2. Risk check: Is exposure within limits?
3. Auditor check: Any red flags?
4. Human review (if required)

### Output
```json
{
  "type": "approval",
  "timestamp": "2026-01-04T12:00:15Z",
  "proposal_id": "prop_12345",
  "status": "approved",
  "checks": {
    "policy": {"passed": true},
    "risk": {"passed": true, "exposure": 0.02},
    "auditor": {"passed": true}
  },
  "approver": "approval_service",
  "valid_until": "2026-01-04T12:05:00Z"
}
```

## Stage 5: Execute

### Input
- Approved proposal
- Execution parameters

### Process
1. Executor receives approved proposal
2. Transaction/action is prepared
3. Execution is attempted
4. Result is captured

### Output
```json
{
  "type": "execution",
  "timestamp": "2026-01-04T12:00:20Z",
  "agent_id": "executor_019",
  "proposal_id": "prop_12345",
  "result": {
    "status": "success",
    "tx_hash": "0x...",
    "gas_used": 150000,
    "execution_time_ms": 2500
  }
}
```

## Stage 6: Verify

### Input
- Execution result
- Expected outcome

### Process
1. Auditor verifies execution matches proposal
2. Outcome is recorded
3. Reputation is updated
4. Rewards/slashing applied

### Output
```json
{
  "type": "verification",
  "timestamp": "2026-01-04T12:00:25Z",
  "agent_id": "auditor_003",
  "execution_id": "exec_67890",
  "verified": true,
  "discrepancies": [],
  "reputation_delta": {
    "executor_019": +0.01,
    "analyst_042": +0.02
  }
}
```

## Data Storage

| Data Type | Storage Location | Retention |
|-----------|------------------|-----------|
| Observations | Off-chain DB | 30 days |
| Analyses | Off-chain DB | 90 days |
| Proposals | Off-chain + L3 hash | Permanent |
| Approvals | Off-chain + L3 hash | Permanent |
| Executions | L3 (if on-chain) | Permanent |
| Verifications | Off-chain + L3 hash | Permanent |

## Error Handling

Each stage has defined error paths:

```
┌─────────┐     ┌─────────┐     ┌─────────┐
│ Observe │ ──► │ Analyze │ ──► │ Propose │
└────┬────┘     └────┬────┘     └────┬────┘
     │               │               │
     ▼               ▼               ▼
┌─────────┐     ┌─────────┐     ┌─────────┐
│  Retry  │     │  Retry  │     │  Retry  │
│   or    │     │   or    │     │   or    │
│  Alert  │     │  Alert  │     │  Alert  │
└─────────┘     └─────────┘     └─────────┘
```

Failures at any stage are logged and may trigger alerts or automatic retries.

