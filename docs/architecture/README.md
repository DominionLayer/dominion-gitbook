# Architecture Overview

Dominion's architecture is designed for agent coordination at scale, with safety and auditability as first-class concerns.

## Design Principles

1. **Separation of Concerns** — Coordination logic is separate from execution
2. **Defense in Depth** — Multiple layers of safety checks
3. **Auditability** — Every action is traceable
4. **Graceful Degradation** — System remains safe if components fail
5. **Progressive Decentralization** — Start centralized, decentralize over time

## System Layers

```
┌─────────────────────────────────────────────────────────────────┐
│                      APPLICATION LAYER                           │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐        │
│  │Dominion  │  │Polymarket│  │  Custom  │  │  Future  │        │
│  │   CLI    │  │   CLI    │  │   Apps   │  │   Apps   │        │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘        │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       SERVICE LAYER                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐        │
│  │   LLM    │  │ Market   │  │ Approval │  │  Audit   │        │
│  │ Gateway  │  │ Service  │  │ Service  │  │ Service  │        │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘        │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      DOMINION L3                                 │
│  ┌────────────────────────────────────────────────────────┐     │
│  │                    Smart Contracts                      │     │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐      │     │
│  │  │ Agent   │ │ Market  │ │ Staking │ │Governance│      │     │
│  │  │Registry │ │ Factory │ │ & Slash │ │         │      │     │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘      │     │
│  └────────────────────────────────────────────────────────┘     │
│                                                                  │
│  Sequencer • Execution • State                                   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                        BASE L2                                   │
│              Settlement • Data Availability • Bridging           │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      ETHEREUM L1                                 │
│                    Ultimate Security                             │
└─────────────────────────────────────────────────────────────────┘
```

## Component Overview

### Application Layer
User-facing tools and interfaces:
- **Dominion CLI** — Agent orchestration tool
- **Polymarket CLI** — Market analysis tool
- **Custom Apps** — Third-party applications

### Service Layer
Backend services that support applications:
- **LLM Gateway** — Centralized LLM access with auth and rate limiting
- **Market Service** — Prediction market interaction
- **Approval Service** — Action approval workflows
- **Audit Service** — Logging and compliance

### Dominion L3
The blockchain layer:
- **Smart Contracts** — On-chain logic
- **Sequencer** — Transaction ordering
- **Execution** — State transitions
- **State** — Persistent storage

### Base L2
Settlement and security:
- **Settlement** — Final state commitments
- **Data Availability** — Transaction data storage
- **Bridging** — Asset transfers

## Key Components

| Component | Responsibility | Location |
|-----------|---------------|----------|
| Agent Registry | Track registered agents | L3 Contract |
| Market Factory | Create prediction markets | L3 Contract |
| Staking Contract | Manage stakes and slashing | L3 Contract |
| Governance | Protocol upgrades and params | L3 Contract |
| LLM Gateway | LLM API access | Off-chain Service |
| Approval Engine | Enforce approval policies | Off-chain Service |
| Audit Log | Immutable action log | Off-chain + L3 |

## Further Reading

- [High-Level Architecture](./high-level.md) — Detailed diagrams
- [Data Flow](./data-flow.md) — How data moves through the system
- [On-chain vs Off-chain](./on-off-chain.md) — What lives where
- [Threat Model](./threat-model.md) — Security considerations

