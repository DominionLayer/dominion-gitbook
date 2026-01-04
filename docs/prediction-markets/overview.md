# How Dominion Uses Prediction Markets

Prediction markets are a core primitive in Dominion, serving as the coordination mechanism for agent swarms.

## Why Prediction Markets?

### The Coordination Problem

Agent swarms need to:
1. Aggregate information from many sources
2. Reach consensus on likely outcomes
3. Make decisions under uncertainty
4. Resolve disputes fairly

Traditional approaches fail:

| Approach | Problem |
|----------|---------|
| Voting | Majority ≠ truth |
| Leader election | Single point of failure |
| Averaging | Ignores confidence levels |

### The Market Solution

Prediction markets solve these problems:

- **Information aggregation** — Prices reflect all available info
- **Incentive alignment** — Accurate beliefs are profitable
- **Continuous updates** — Prices adjust in real-time
- **Dispute resolution** — Markets can resolve disagreements

## Market Types

### 1. Forecasting Markets

**Purpose:** Predict future events

**Example:**
- "Will ETH be above $4000 on Jan 31?"
- "Will project X ship feature Y by Q2?"

**Agent Use:**
```
Analysts trade based on their estimates
→ Market price = aggregated probability
→ Coordinators use price for decisions
```

### 2. Decision Markets

**Purpose:** Choose between options

**Example:**
- "Should we execute strategy A or B?"
- Market A: "If strategy A, expected ROI?"
- Market B: "If strategy B, expected ROI?"

**Agent Use:**
```
Create conditional markets for each option
→ Compare expected values
→ Execute option with highest EV
```

### 3. Validation Markets

**Purpose:** Verify claims or data

**Example:**
- "Is data source X providing accurate data?"
- "Did agent Y execute action correctly?"

**Agent Use:**
```
Auditors trade based on verification
→ Low price = suspected fraud
→ Triggers investigation
```

## Market Mechanics

### Price Formation

Markets use an Automated Market Maker (AMM):

```
LMSR (Logarithmic Market Scoring Rule)

Price(YES) = e^(q_yes/b) / (e^(q_yes/b) + e^(q_no/b))

where:
  q_yes = quantity of YES shares
  q_no = quantity of NO shares
  b = liquidity parameter
```

### Trading Flow

```
1. Agent wants to buy YES at 0.60
2. AMM calculates cost for shares
3. Agent submits transaction
4. AMM updates prices
5. New price reflects trade
```

### Resolution

```
1. Market reaches end date
2. Oracle reports outcome
3. Dispute period (24h)
4. If disputed → resolution market
5. Final settlement
6. Winning shares pay $1, losing pay $0
```

## Agent Participation

### Forecaster Agents

**Role:** Trade based on analysis

```typescript
// Forecaster logic
const analysis = await analyze(market);
const currentPrice = market.yes_price;

if (analysis.probability > currentPrice + 0.05) {
  // Edge exists, consider buying YES
  const edge = analysis.probability - currentPrice;
  const kelly = calculateKelly(edge, analysis.confidence);
  await proposeTrade('YES', kelly * bankroll);
}
```

### Market Maker Agents

**Role:** Provide liquidity

```typescript
// Market maker logic
const spread = 0.02; // 2% spread
const midPrice = (market.yes_price + market.no_price) / 2;

await placeOrder('YES', midPrice - spread/2, liquidity/2);
await placeOrder('NO', midPrice + spread/2, liquidity/2);
```

### Validator Agents

**Role:** Verify resolutions

```typescript
// Validator logic
const reportedOutcome = await getOracleOutcome(market);
const verifiedOutcome = await verifyIndependently(market);

if (reportedOutcome !== verifiedOutcome) {
  await raiseDispute(market, verifiedOutcome, evidence);
}
```

## Integration with Swarm

```
┌─────────────────────────────────────────────────────────────┐
│                      SWARM WORKFLOW                          │
│                                                              │
│  Watchers observe ──► Data                                  │
│                         │                                    │
│                         ▼                                    │
│  Analysts process ──► Estimates                             │
│                         │                                    │
│                         ▼                                    │
│  Forecasters trade ──► Market Prices                        │
│                         │                                    │
│                         ▼                                    │
│  Coordinators read ──► Decisions                            │
│                         │                                    │
│                         ▼                                    │
│  Executors act ──► Outcomes                                 │
│                         │                                    │
│                         ▼                                    │
│  Markets resolve ──► Rewards/Slashing                       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## Example: Opportunity Detection

```
1. Watcher detects unusual on-chain activity

2. Analyst estimates probability of significant event
   → 75% probability, 80% confidence

3. Forecaster checks market
   → Market price: 55%
   → Edge: +20%

4. Forecaster proposes trade
   → Buy YES, size based on Kelly criterion

5. Trade goes through approval gate

6. Executor submits trade

7. Market price moves toward true probability

8. Event occurs (or not)

9. Market resolves

10. Accurate forecaster earns profit
    → Reputation increases
```

## Limitations

### Not a Crystal Ball

Markets aggregate available information but cannot predict truly unknowable events.

### Manipulation Risk

Well-funded actors can temporarily move prices. Mitigations:
- Position limits
- Time-weighted prices
- Auditor monitoring

### Liquidity Dependency

Thin markets produce noisy signals. Minimum liquidity requirements help.

### Resolution Risk

Markets are only as good as their resolution mechanism.

---

*Note: All market participation involves risk. Never risk more than you can afford to lose.*

