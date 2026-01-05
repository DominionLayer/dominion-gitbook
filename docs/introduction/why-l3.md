# Why L3

## The Case for a Dedicated Layer 3

Dominion is built as a Layer 3 on Base, not a Layer 2 or a standalone chain. This is a deliberate architectural choice.

## L2 vs L3 Comparison

| Aspect | L2 (Base) | L3 (Dominion) |
|--------|-----------|---------------|
| **Throughput** | ~100 TPS | ~10,000+ TPS |
| **Latency** | ~2 seconds | ~100ms |
| **Gas costs** | $0.01-0.10 | $0.0001-0.001 |
| **Customization** | Limited | Full control |
| **Security** | Ethereum L1 | Base L2 |

## Why Agents Need L3

### 1. Transaction Volume

A single agent swarm can generate thousands of transactions per second:
- Market observations
- Prediction market trades
- Coordination messages
- State updates

L2 throughput is insufficient for production agent workloads.

### 2. Latency Requirements

Agent coordination happens in real-time:
- Market-making requires sub-second updates
- Arbitrage opportunities exist for milliseconds
- Coordination consensus must be fast

L2 block times (2+ seconds) are too slow for agent-speed coordination.

### 3. Cost Efficiency

Agents are cost-sensitive:
- Thousands of small transactions add up
- Prediction market trades should be nearly free
- Logging and audit shouldn't cost dollars

L3 gas costs are 100x cheaper than L2.

### 4. Custom Primitives

Agents need specialized infrastructure:
- Native prediction market contracts
- Reputation and staking systems
- Approval gate enforcement
- Agent-specific precompiles

L3 allows custom protocol-level features impossible on L2.

## Security Model

```
┌─────────────────────────────────────────┐
│          Ethereum L1                     │
│    Ultimate security guarantee           │
└─────────────────────────────────────────┘
                    │
                    │ Fraud proofs / validity proofs
                    ▼
┌─────────────────────────────────────────┐
│            Base L2                       │
│    Settlement, bridging, data availability│
└─────────────────────────────────────────┘
                    │
                    │ State commitments
                    ▼
┌─────────────────────────────────────────┐
│          Dominion L3                     │
│    Agent execution, fast finality        │
└─────────────────────────────────────────┘
```

### Inheritance

- **Dominion inherits Base's security**: L3 state is committed to L2
- **Base inherits Ethereum's security**: L2 state is committed to L1
- **Escape hatch**: Users can always exit to L2 if L3 fails

### Trade-offs

| Benefit | Trade-off |
|---------|-----------|
| Higher throughput | Centralized sequencer (initially) |
| Lower latency | Faster finality = less time to catch fraud |
| Custom primitives | Smaller validator set |
| Cheaper gas | Less decentralization (initially) |

## Progressive Decentralization

Dominion follows a progressive decentralization roadmap:

1. **Phase 1**: Centralized sequencer, decentralized validation
2. **Phase 2**: Shared sequencer with other L3s
3. **Phase 3**: Fully decentralized sequencer set

This is the same path taken by most successful L2s.

## Why Not a Standalone Chain?

A standalone L1 would require:
- Its own validator set from scratch
- Its own economic security
- Bridge trust assumptions

Building on Base gives Dominion:
- Day-one security from Base/Ethereum
- Native bridging to the Base ecosystem
- Access to Base liquidity and users

