# Safety & Guardrails

Prediction markets involve financial risk. Dominion implements multiple safety measures.

## Core Safety Principles

1. **No Guaranteed Profits** — Never claim sure outcomes
2. **Uncertainty is Real** — Always show confidence ranges
3. **Human Oversight** — Humans approve significant actions
4. **Gradual Exposure** — Start small, scale with trust

## Mandatory Disclaimers

Every market-related output must include:

```
⚠️ DISCLAIMER
This is analysis and simulation only, NOT financial advice.
Past performance does not guarantee future results.
Never risk more than you can afford to lose.
```

### In Code

```typescript
const DISCLAIMER = `
This is analysis and simulation, not financial advice.
Confidence: ${analysis.confidence * 100}% (±${uncertaintyBand}%)
`;

function formatOutput(analysis: Analysis): string {
  return `
${analysis.summary}

${DISCLAIMER}
  `;
}
```

## Confidence & Uncertainty

### Always Show Ranges

```
❌ Bad: "Probability: 65%"
✅ Good: "Probability: 65% (confidence: 72%, range: 55-75%)"
```

### Confidence Calculation

```typescript
interface AnalysisOutput {
  estimatedProbability: number;  // Point estimate
  confidence: number;            // How sure (0-1)
  range: {
    low: number;                 // Pessimistic
    high: number;                // Optimistic
  };
  uncertaintySources: string[];  // What could be wrong
}

function calculateRange(prob: number, confidence: number): Range {
  const band = (1 - confidence) * 0.3; // Scale uncertainty
  return {
    low: Math.max(0, prob - band),
    high: Math.min(1, prob + band),
  };
}
```

### Displaying Uncertainty

```
Market Analysis: Will ETH > $4000?
═══════════════════════════════════

Point Estimate: 65%
Confidence: 72%

Scenario Range:
  Pessimistic: 55%
  Base Case:   65%
  Optimistic:  75%

Uncertainty Sources:
  • Macro conditions unpredictable
  • Limited historical precedent
  • Model trained on different market regime

⚠️ This is analysis only, not financial advice.
```

## Position Limits

### Per-Agent Limits

```typescript
const POSITION_LIMITS = {
  // Max position in single market
  maxPositionPerMarket: 0.10,  // 10% of bankroll
  
  // Max total exposure
  maxTotalExposure: 0.50,      // 50% of bankroll
  
  // Max single trade
  maxSingleTrade: 0.05,        // 5% of bankroll
  
  // Max correlated exposure
  maxCorrelatedExposure: 0.20, // 20% in correlated markets
};

function validateTrade(trade: Trade, portfolio: Portfolio): ValidationResult {
  const newPosition = portfolio.getPosition(trade.market) + trade.size;
  const newExposure = portfolio.totalExposure + trade.size;
  
  if (newPosition > POSITION_LIMITS.maxPositionPerMarket * portfolio.bankroll) {
    return { valid: false, reason: 'Position limit exceeded' };
  }
  
  if (newExposure > POSITION_LIMITS.maxTotalExposure * portfolio.bankroll) {
    return { valid: false, reason: 'Total exposure limit exceeded' };
  }
  
  return { valid: true };
}
```

### Protocol-Level Limits

| Limit | Value | Purpose |
|-------|-------|---------|
| Max position per market | 10% of pool | Prevent manipulation |
| Max daily volume | Varies | Prevent wash trading |
| Min liquidity | 1000 DOM | Ensure price stability |

## Approval Gates

### Trade Approval Flow

```
Trade Proposal
     │
     ▼
┌─────────────────┐
│  Policy Check   │ ← Position limits, allowed markets
└────────┬────────┘
         │ Pass
         ▼
┌─────────────────┐
│   Risk Check    │ ← Exposure, correlation, VaR
└────────┬────────┘
         │ Pass
         ▼
┌─────────────────┐
│  Rate Limiter   │ ← Max trades per minute
└────────┬────────┘
         │ Pass
         ▼
┌─────────────────┐
│ Human Review?   │ ← If size > threshold
└────────┬────────┘
         │ Approved
         ▼
    Execute Trade
```

### Configuration

```yaml
# dominion.config.yaml
safety:
  # Require approval for trades
  require_approval: true
  
  # Threshold for human review
  human_review_threshold: 1000  # DOM
  
  # Rate limits
  max_trades_per_minute: 10
  
  # Auto-reject conditions
  auto_reject:
    - market_liquidity_below: 500
    - price_impact_above: 0.05
    - correlation_above: 0.8
```

## Manipulation Prevention

### Detection Methods

```typescript
class ManipulationDetector {
  // Detect wash trading (self-trading)
  async detectWashTrading(trades: Trade[]): Promise<Alert | null> {
    const graph = this.buildTradeGraph(trades);
    const cycles = this.findCycles(graph);
    
    if (cycles.length > 0) {
      return {
        type: 'WASH_TRADING',
        severity: 'HIGH',
        evidence: cycles,
      };
    }
    return null;
  }
  
  // Detect price manipulation
  async detectPriceManipulation(market: Market): Promise<Alert | null> {
    const priceHistory = await this.getPriceHistory(market, '1h');
    const volatility = this.calculateVolatility(priceHistory);
    const expectedVol = this.getExpectedVolatility(market);
    
    if (volatility > expectedVol * 3) {
      return {
        type: 'PRICE_MANIPULATION',
        severity: 'MEDIUM',
        evidence: { volatility, expected: expectedVol },
      };
    }
    return null;
  }
  
  // Detect coordinated trading
  async detectCoordination(trades: Trade[]): Promise<Alert | null> {
    const timing = this.analyzeTimingPatterns(trades);
    const clustering = this.detectClustering(timing);
    
    if (clustering.score > 0.8) {
      return {
        type: 'COORDINATED_TRADING',
        severity: 'MEDIUM',
        evidence: clustering,
      };
    }
    return null;
  }
}
```

### Response Actions

| Alert Type | Auto-Response | Manual Review |
|------------|--------------|---------------|
| Wash trading | Flag trades, pause agent | Investigate, potential ban |
| Price manipulation | Widen spreads, pause market | Review, potential resolution |
| Coordinated trading | Increase monitoring | Investigate relationships |

## Forbidden Language

The following phrases must NEVER appear in outputs:

```typescript
const FORBIDDEN_PHRASES = [
  'guaranteed profit',
  'risk-free',
  'sure win',
  'can\'t lose',
  'certain return',
  'free money',
  'always profitable',
  'no downside',
];

function validateOutput(text: string): boolean {
  for (const phrase of FORBIDDEN_PHRASES) {
    if (text.toLowerCase().includes(phrase)) {
      throw new Error(`Forbidden phrase detected: "${phrase}"`);
    }
  }
  return true;
}
```

## Emergency Procedures

### Market Pause

```typescript
// Conditions that trigger market pause
const PAUSE_CONDITIONS = {
  priceImpact: 0.20,        // 20% in 1 minute
  volumeSpike: 10,          // 10x normal
  oracleFailure: true,
  manipulationAlert: true,
};

async function checkPauseConditions(market: Market): Promise<boolean> {
  const impact = await calculateRecentPriceImpact(market);
  if (impact > PAUSE_CONDITIONS.priceImpact) {
    await pauseMarket(market, 'PRICE_IMPACT');
    return true;
  }
  // ... other checks
  return false;
}
```

### Agent Suspension

```typescript
// Conditions for agent suspension
const SUSPENSION_TRIGGERS = {
  manipulationDetected: true,
  repeatedPolicyViolations: 3,
  slashingEvents: 2,
  auditFailures: 5,
};
```

## Audit Trail

All market interactions are logged:

```typescript
interface MarketAuditEntry {
  timestamp: string;
  marketId: string;
  action: 'TRADE' | 'CREATE' | 'RESOLVE' | 'DISPUTE';
  agentId: string;
  details: Record<string, unknown>;
  approvals: string[];
  outcome: 'SUCCESS' | 'FAILED' | 'REJECTED';
}
```

Logs are:
- Immutable (hash-chained)
- Queryable (for investigations)
- Anchored on-chain (for disputes)

