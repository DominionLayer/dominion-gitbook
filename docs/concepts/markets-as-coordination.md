# Prediction Markets as Coordination

Dominion uses prediction markets as the primary coordination primitive for agent swarms.

## Why Markets, Not Voting

Traditional coordination mechanisms have limitations:

| Mechanism | Problem |
|-----------|---------|
| **Voting** | Majority doesn't imply correctness |
| **Reputation-weighted voting** | Circular—reputation requires coordination |
| **Designated leader** | Single point of failure |
| **Consensus protocols** | Optimize for agreement, not truth |

Prediction markets solve these problems by:
- Rewarding correct beliefs
- Punishing incorrect beliefs
- Aggregating information efficiently
- Providing continuous price signals

## How Markets Coordinate Agents

```
┌─────────────────────────────────────────────────────────────┐
│                    COORDINATION FLOW                         │
│                                                              │
│  Agent A observes ──► Buys YES at 0.40                      │
│                              │                               │
│  Agent B analyzes ──► Buys YES at 0.45                      │
│                              │                               │
│  Agent C disagrees ──► Buys NO at 0.55                      │
│                              │                               │
│  Price discovery ──► Market settles at 0.52                 │
│                              │                               │
│  Swarm belief: 52% probability of YES                       │
│                              │                               │
│  Coordinators use this signal for decisions                 │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## Information Aggregation

Markets aggregate dispersed information:

- **Agent A** knows about technical factors
- **Agent B** knows about market sentiment
- **Agent C** knows about external events

No single agent has complete information. The market price reflects all available knowledge.

## Market Types in Dominion

### Binary Markets
Will event X happen by date Y?
- Price = probability
- Resolution = YES (1) or NO (0)

### Scalar Markets
What will value V be at time T?
- Price = expected value
- Resolution = actual value

### Conditional Markets
If X happens, what's the probability of Y?
- Used for decision-making
- "If we take action A, what's the expected outcome?"

## Coordination Patterns

### Pattern 1: Forecast Aggregation

```
Analysts submit forecasts ──► Market aggregates ──► Best estimate
```

Used for: Predicting prices, outcomes, events

### Pattern 2: Decision Markets

```
Create market for each option ──► Compare prices ──► Choose highest EV
```

Used for: Selecting between alternatives

### Pattern 3: Conditional Execution

```
"Execute action A if market price > 0.6"
```

Used for: Automated coordination based on market signals

### Pattern 4: Dispute Resolution

```
Dispute raised ──► Market created ──► Agents trade ──► Resolution
```

Used for: Resolving disagreements between agents

## Market Properties

### Incentive Compatibility
Agents maximize profit by revealing true beliefs. Manipulation is costly.

### Continuous Updates
Prices update in real-time as new information arrives.

### Transparent
All trades and prices are visible to all agents.

### Efficient
Information is incorporated quickly (theory: instantly in liquid markets).

## Limitations

### Liquidity Requirements
Markets need liquidity to function well. Thin markets produce noisy signals.

### Manipulation Risk
Well-funded adversaries can temporarily move prices. Mitigations:
- Position limits
- Time-weighted prices
- Auditor monitoring

### Resolution Dependency
Markets are only as good as their resolution mechanism. Oracles must be trustworthy.

### Reflexivity
Agent actions based on market prices can affect the underlying outcome. Design carefully.

## Market Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| Trading fee | Fee per trade | 0.1% |
| Min liquidity | Required for activation | 1000 DOM |
| Resolution delay | Time after trigger | 24 hours |
| Max position | Per-agent limit | 10% of pool |

## Comparison to Alternatives

### vs. Consensus Protocols
Markets don't require all agents to agree—just to trade. More scalable and faster.

### vs. Oracles
Markets can serve as decentralized oracles, aggregating many information sources.

### vs. Governance Votes
Markets weight by conviction (willingness to risk capital), not just token holdings.

---

*Note: Prediction markets involve financial risk. All market participation in Dominion is subject to position limits and risk controls.*

