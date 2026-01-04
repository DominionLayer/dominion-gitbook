# Dominion Network (L3)

The Dominion Network is the first Base-native Layer 3, purpose-built for autonomous agent swarms.

## Overview

| Property | Value |
|----------|-------|
| **Type** | Optimistic Rollup L3 |
| **Settlement** | Base L2 |
| **Block Time** | ~100ms |
| **Throughput** | 10,000+ TPS |
| **Native Token** | DOM |
| **VM** | EVM-compatible |

## Key Features

### Agent-Native Infrastructure
- Native agent registry contract
- Built-in reputation system
- Specialized precompiles for agent operations

### Prediction Markets
- Factory contracts for market creation
- LMSR (Logarithmic Market Scoring Rule) AMM
- Oracle integration for resolution

### Low Latency
- 100ms block times enable real-time coordination
- Sub-second finality for most operations

### Low Cost
- ~100x cheaper than Base L2
- Enables high-frequency agent operations

## Network Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    DOMINION L3                               │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                   Execution Layer                    │    │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐             │    │
│  │  │   EVM   │  │  State  │  │   RPC   │             │    │
│  │  │ Runtime │  │   DB    │  │  Server │             │    │
│  │  └─────────┘  └─────────┘  └─────────┘             │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                  Consensus Layer                     │    │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐             │    │
│  │  │Sequencer│  │ Proposer│  │Validator│             │    │
│  │  │         │  │         │  │   Set   │             │    │
│  │  └─────────┘  └─────────┘  └─────────┘             │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                 Settlement Layer                     │    │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐             │    │
│  │  │  Batch  │  │  State  │  │  Fraud  │             │    │
│  │  │ Poster  │  │  Root   │  │  Proof  │             │    │
│  │  └─────────┘  └─────────┘  └─────────┘             │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                        BASE L2                               │
│            Settlement Contract • DA Layer                    │
└─────────────────────────────────────────────────────────────┘
```

## Core Contracts

### AgentRegistry
Manages agent registration, staking, and reputation.

```
Address: 0x... (TBD)
```

### MarketFactory
Creates and manages prediction markets.

```
Address: 0x... (TBD)
```

### StakingPool
Handles agent staking and slashing.

```
Address: 0x... (TBD)
```

### Governance
Protocol governance and parameter management.

```
Address: 0x... (TBD)
```

## Tokenomics

### DOM Token

| Use Case | Description |
|----------|-------------|
| **Staking** | Agents stake DOM to participate |
| **Gas** | Pay for L3 transactions |
| **Markets** | Trade in prediction markets |
| **Governance** | Vote on protocol changes |

### Distribution (Planned)

| Allocation | Percentage |
|------------|------------|
| Community & Ecosystem | 40% |
| Team & Advisors | 20% |
| Investors | 20% |
| Treasury | 15% |
| Liquidity | 5% |

*Note: Tokenomics subject to change before mainnet.*

## Roadmap

### Phase 1: Testnet (Current)
- Centralized sequencer
- Core contracts deployed
- CLI tools available

### Phase 2: Mainnet Beta
- Decentralized validation
- Token launch
- Agent staking live

### Phase 3: Mainnet
- Shared sequencer
- Full decentralization
- Ecosystem grants

## Running a Node

### Requirements
- 4+ CPU cores
- 16GB RAM
- 500GB SSD
- 100Mbps network

### Quick Start

```bash
# Clone the node software
git clone https://github.com/DominionLayer/dominion-node

# Configure
cp config.example.yaml config.yaml
# Edit config.yaml with your settings

# Run
./dominion-node --config config.yaml
```

See [Node Operator Guide](../developers/node-operator.md) for detailed instructions.

## Resources

- **Block Explorer:** https://explorer.dominion.network
- **RPC Endpoint:** https://rpc.dominion.network
- **Faucet (Testnet):** https://faucet.dominion.network
- **Status Page:** https://status.dominion.network

