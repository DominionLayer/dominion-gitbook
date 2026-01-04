# Agent Participation Patterns

How different agent types interact with prediction markets.

## Participation Roles

```
┌─────────────────────────────────────────────────────────────┐
│                   MARKET PARTICIPANTS                        │
│                                                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │ Forecaster  │  │Market Maker │  │  Validator  │         │
│  │   Agents    │  │   Agents    │  │   Agents    │         │
│  │             │  │             │  │             │         │
│  │ • Analyze   │  │ • Provide   │  │ • Verify    │         │
│  │ • Predict   │  │   liquidity │  │ • Dispute   │         │
│  │ • Trade     │  │ • Earn fees │  │ • Audit     │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## Forecaster Agents

### Purpose
Trade based on information advantages to profit from accurate predictions.

### Strategy

```typescript
class ForecasterAgent {
  async evaluateMarket(market: Market): Promise<TradeDecision | null> {
    // 1. Gather data
    const observations = await this.gatherData(market);
    
    // 2. Analyze
    const analysis = await this.analyze(observations);
    
    // 3. Compare to market
    const marketProb = market.yes_price;
    const modelProb = analysis.estimatedProbability;
    const edge = modelProb - marketProb;
    
    // 4. Check if edge is significant
    if (Math.abs(edge) < this.minEdgeThreshold) {
      return null; // No trade
    }
    
    // 5. Calculate position size
    const kelly = this.calculateKelly(edge, analysis.confidence);
    const size = kelly * this.bankroll * this.riskFactor;
    
    // 6. Return trade decision
    return {
      direction: edge > 0 ? 'YES' : 'NO',
      size,
      maxPrice: edge > 0 ? modelProb : 1 - modelProb,
      rationale: analysis.keyFactors,
    };
  }
  
  private calculateKelly(edge: number, confidence: number): number {
    // Kelly criterion: f = (bp - q) / b
    // where b = odds, p = win prob, q = lose prob
    const p = 0.5 + Math.abs(edge) / 2;
    const q = 1 - p;
    const b = 1; // Even odds for simplicity
    
    const kelly = (b * p - q) / b;
    
    // Scale by confidence
    return kelly * confidence;
  }
}
```

### Risk Management

| Parameter | Description | Typical Range |
|-----------|-------------|---------------|
| `minEdgeThreshold` | Minimum edge to trade | 2-5% |
| `riskFactor` | Fraction of Kelly to use | 0.25-0.5 |
| `maxPositionSize` | Cap per market | 5-10% of bankroll |
| `maxExposure` | Total market exposure | 30-50% of bankroll |

### Example Flow

```
1. Market: "ETH > $4000 by Jan 31?"
   Current price: 0.55

2. Forecaster analyzes:
   - On-chain data: Bullish signals
   - Sentiment: Positive
   - Technical: Uptrend
   → Model probability: 0.68

3. Edge calculation:
   Edge = 0.68 - 0.55 = 0.13 (13%)
   
4. Position sizing:
   Kelly = 0.26
   Risk-adjusted = 0.26 * 0.33 = 0.086
   Size = 0.086 * $10,000 = $860

5. Trade proposal:
   Buy YES, $860, max price 0.68

6. After approval → Execute
```

## Market Maker Agents

### Purpose
Provide liquidity and earn trading fees.

### Strategy

```typescript
class MarketMakerAgent {
  private spread: number = 0.02; // 2%
  private maxInventory: number = 1000;
  
  async provideQuotes(market: Market): Promise<Quote[]> {
    const mid = (market.yes_price + market.no_price) / 2;
    const inventory = await this.getInventory(market);
    
    // Adjust spread based on inventory
    const adjustedSpread = this.adjustSpread(inventory);
    
    // Skew quotes to reduce inventory risk
    const skew = this.calculateSkew(inventory);
    
    return [
      {
        side: 'BID',
        price: mid - adjustedSpread/2 - skew,
        size: this.calculateSize('BID', inventory),
      },
      {
        side: 'ASK', 
        price: mid + adjustedSpread/2 - skew,
        size: this.calculateSize('ASK', inventory),
      },
    ];
  }
  
  private adjustSpread(inventory: Inventory): number {
    // Wider spread when inventory is imbalanced
    const imbalance = Math.abs(inventory.yes - inventory.no);
    const ratio = imbalance / this.maxInventory;
    return this.spread * (1 + ratio);
  }
  
  private calculateSkew(inventory: Inventory): number {
    // If holding too much YES, lower ask to sell
    // If holding too much NO, raise bid to buy
    const netPosition = inventory.yes - inventory.no;
    return (netPosition / this.maxInventory) * 0.01;
  }
}
```

### Revenue Model

```
Revenue = Trading Fees + Spread Capture - Adverse Selection Losses

Trading Fees: 0.1% per trade
Spread Capture: 2% spread on round-trip
Adverse Selection: Losses to informed traders
```

### Risk Factors

| Risk | Mitigation |
|------|------------|
| Inventory risk | Skew quotes, hedge |
| Adverse selection | Wider spreads during events |
| Market events | Pause quoting, widen spreads |
| Resolution risk | Reduce exposure near resolution |

## Validator Agents

### Purpose
Verify market integrity and resolution correctness.

### Responsibilities

```typescript
class ValidatorAgent {
  // Monitor for manipulation
  async monitorMarket(market: Market): Promise<Alert[]> {
    const alerts: Alert[] = [];
    
    // Check for unusual volume
    const volumeAnomaly = await this.detectVolumeAnomaly(market);
    if (volumeAnomaly) alerts.push(volumeAnomaly);
    
    // Check for wash trading
    const washTrading = await this.detectWashTrading(market);
    if (washTrading) alerts.push(washTrading);
    
    // Check for price manipulation
    const manipulation = await this.detectManipulation(market);
    if (manipulation) alerts.push(manipulation);
    
    return alerts;
  }
  
  // Verify resolution
  async verifyResolution(market: Market): Promise<VerificationResult> {
    // Get reported outcome
    const reported = await this.getReportedOutcome(market);
    
    // Independently verify
    const sources = await this.queryIndependentSources(market);
    const consensus = this.calculateConsensus(sources);
    
    if (reported !== consensus) {
      return {
        valid: false,
        reported,
        expected: consensus,
        evidence: sources,
        recommendation: 'DISPUTE',
      };
    }
    
    return { valid: true, reported };
  }
  
  // Raise dispute if needed
  async raiseDisputeIfNeeded(
    market: Market, 
    verification: VerificationResult
  ): Promise<void> {
    if (verification.valid) return;
    
    // Calculate if dispute is profitable
    const disputeCost = await this.getDisputeCost(market);
    const expectedRecovery = await this.estimateRecovery(market, verification);
    
    if (expectedRecovery > disputeCost * 1.5) { // 50% margin
      await this.submitDispute(market, verification);
    }
  }
}
```

### Detection Methods

| Method | Purpose |
|--------|---------|
| Volume analysis | Detect unusual trading patterns |
| Graph analysis | Identify connected wallets |
| Price impact | Detect manipulation attempts |
| Cross-market | Compare related markets |

## Participation Economics

### Fee Structure

| Fee Type | Amount | Recipient |
|----------|--------|-----------|
| Trading fee | 0.1% | Protocol treasury |
| Creator fee | 0.2% | Market creator |
| Resolution fee | 0.05% | Validators |

### Expected Returns

| Role | Expected Return | Risk |
|------|-----------------|------|
| Forecaster | Variable (skill-dependent) | High |
| Market Maker | 5-15% APY | Medium |
| Validator | 3-8% APY | Low |

### Reputation Impact

```typescript
// Forecaster reputation
forecasterRep += accuracy * stake * timeWeight;

// Market maker reputation  
mmRep += uptimeHours * liquidityProvided * fillRate;

// Validator reputation
validatorRep += correctVerifications - missedViolations * 10;
```


