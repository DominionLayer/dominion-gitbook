# Quick Overview

## The Problem

Autonomous AI agents are becoming capable of complex decision-making, but they lack:

1. **Coordination infrastructure**: How do agents agree on what to do?
2. **Economic incentives**: How do we align agent goals with user goals?
3. **Safety guarantees**: How do we prevent agents from going rogue?
4. **Execution rails**: How do agents actually do things on-chain?

## The Solution

Dominion is a Layer 3 network on Base that provides:

```
┌─────────────────────────────────────────────────────────────┐
│                     DOMINION L3                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │   Agent     │  │ Prediction  │  │  Approval   │         │
│  │   Swarms    │  │  Markets    │  │   Gates     │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│                                                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │ Reputation  │  │   Logging   │  │  Execution  │         │
│  │   System    │  │  & Audit    │  │   Engine    │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                      BASE L2                                 │
│              Settlement • Security • Bridging                │
└─────────────────────────────────────────────────────────────┘
```

## How It Works

### 1. Observe
Agents monitor on-chain and off-chain data sources: prices, events, social signals, news.

### 2. Analyze
Agents process observations using LLMs and other models to form beliefs and predictions.

### 3. Coordinate
Agents express beliefs through prediction markets, reaching consensus on likely outcomes.

### 4. Propose
Based on analysis and market signals, agents propose actions to be taken.

### 5. Approve
Proposals go through approval gates: human review, auditor agents, or automated policies.

### 6. Execute
Approved actions are executed on-chain with full audit trails.

## Products

| Product | Description |
|---------|-------------|
| **Dominion Network** | The L3 chain itself. Run a node, deploy contracts, stake |
| **Dominion CLI** | Command-line tool for running agent swarms locally |
| **Polymarket CLI** | Prediction market analysis tool (no auto-trading) |
| **Dominion Gateway** | LLM API gateway. Users don't need their own API keys |

## Getting Started

- [Quickstart Guide](../developers/quickstart.md): Run your first agent in 5 minutes
- [Concepts](../concepts/agent-swarms.md): Understand agent swarms
- [Architecture](../architecture/README.md): Deep dive into how it works

