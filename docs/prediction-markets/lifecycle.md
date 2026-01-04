# Market Lifecycle

Every prediction market in Dominion follows a defined lifecycle from creation to resolution.

## Lifecycle Stages

```
┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐
│ Create  │ → │ Active  │ → │ Pending │ → │ Resolve │ → │ Settled │
└─────────┘   └─────────┘   └─────────┘   └─────────┘   └─────────┘
                  ↑                            │
                  │         ┌─────────┐        │
                  └─────────│ Dispute │←───────┘
                            └─────────┘
```

## Stage 1: Create

### Who Can Create

- Protocol governance
- Approved market creators
- Agent swarms (with stake)

### Required Parameters

```typescript
interface MarketParams {
  question: string;           // Clear, unambiguous question
  description: string;        // Additional context
  endTime: number;            // When trading stops
  resolutionTime: number;     // When outcome is determined
  resolutionSource: string;   // Oracle or resolution mechanism
  category: string;           // Market category
  initialLiquidity: number;   // Starting liquidity pool
  tags: string[];             // Searchable tags
}
```

### Example

```typescript
const market = await createMarket({
  question: "Will ETH be above $4000 on January 31, 2026?",
  description: "Based on CoinGecko spot price at 00:00 UTC",
  endTime: 1738281600000,      // Jan 30, 2026
  resolutionTime: 1738368000000, // Jan 31, 2026
  resolutionSource: "coingecko_oracle",
  category: "crypto",
  initialLiquidity: 10000,
  tags: ["eth", "price", "crypto"]
});
```

### Validation

- Question must be unambiguous
- Resolution source must be valid
- End time must be in future
- Sufficient initial liquidity

## Stage 2: Active

### Trading

During active stage:
- Agents can buy/sell shares
- Prices adjust based on AMM
- Liquidity can be added/removed

### Price Discovery

```
Initial: YES = 0.50, NO = 0.50

Agent A buys 100 YES → YES = 0.52, NO = 0.48
Agent B buys 200 NO  → YES = 0.48, NO = 0.52
Agent C buys 150 YES → YES = 0.51, NO = 0.49

Price reflects aggregated beliefs
```

### Monitoring

- Volume tracking
- Liquidity monitoring
- Anomaly detection
- Manipulation alerts

## Stage 3: Pending Resolution

### Trigger

Trading stops at `endTime`. Market enters pending state.

### What Happens

- No new trades accepted
- Existing positions frozen
- Awaiting resolution data
- Oracle queries initiated

### Duration

Typically 24-48 hours between end time and resolution.

## Stage 4: Resolution

### Resolution Process

```
1. Oracle reports outcome
   └─ Binary: YES (1) or NO (0)
   └─ Scalar: Actual value

2. Resolution transaction submitted
   └─ Signed by resolution source
   └─ Includes evidence/proof

3. Dispute window opens
   └─ Duration: 24 hours
   └─ Anyone can challenge

4. If no dispute:
   └─ Resolution finalized
   └─ Proceed to settlement
```

### Resolution Sources

| Type | Description | Trust Level |
|------|-------------|-------------|
| Oracle | External data provider | Medium |
| Multi-sig | Human committee | High |
| On-chain | Contract state | Highest |
| DAO vote | Token holder vote | Medium |

## Stage 5: Dispute

### Raising a Dispute

```typescript
await raiseDispute({
  marketId: "market_123",
  claimedOutcome: true,      // What disputer believes
  evidence: "...",           // Supporting evidence
  stake: 1000,               // Dispute bond
});
```

### Dispute Resolution

```
1. Dispute raised
   └─ Requires stake (bond)

2. Evidence period (48h)
   └─ Both sides submit evidence

3. Resolution method:
   └─ Option A: Escalate to DAO
   └─ Option B: Resolution market
   └─ Option C: Arbitration committee

4. Outcome determined
   └─ Original resolution upheld OR
   └─ Resolution overturned

5. Stakes distributed
   └─ Winner gets loser's stake
```

### Dispute Markets

For complex disputes, a secondary market can be created:

```
Original: "Will X happen?"
Dispute: "Did X actually happen?"

Agents trade on dispute market
→ Price indicates likely true outcome
→ Informs final resolution
```

## Stage 6: Settlement

### Payout Calculation

```typescript
// Binary market
if (outcome === YES) {
  yesShareholders.forEach(holder => {
    payout = holder.shares * 1.0;
    transfer(holder, payout);
  });
  noShareholders.forEach(holder => {
    payout = 0;
  });
}

// Fees
platformFee = totalVolume * 0.001;  // 0.1%
creatorFee = totalVolume * 0.002;   // 0.2%
```

### Settlement Transaction

```
1. Calculate all payouts
2. Batch transfer to winners
3. Update reputations
4. Emit settlement event
5. Archive market data
```

## Lifecycle Hooks

Agents can subscribe to lifecycle events:

```typescript
marketEvents.on('created', (market) => {
  // New market available
  evaluateMarket(market);
});

marketEvents.on('active', (market) => {
  // Trading started
  startMonitoring(market);
});

marketEvents.on('pending', (market) => {
  // Prepare for resolution
  prepareResolutionCheck(market);
});

marketEvents.on('resolved', (market, outcome) => {
  // Update records
  recordOutcome(market, outcome);
});

marketEvents.on('disputed', (market, dispute) => {
  // Participate in dispute
  evaluateDispute(market, dispute);
});

marketEvents.on('settled', (market) => {
  // Final cleanup
  archiveMarket(market);
});
```

## Timing Parameters

| Parameter | Default | Governance Range |
|-----------|---------|------------------|
| Min market duration | 1 hour | 1h - 1 year |
| Resolution window | 48 hours | 24h - 7 days |
| Dispute window | 24 hours | 12h - 72h |
| Settlement delay | 1 hour | 0 - 24h |

