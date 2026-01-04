# Glossary

Technical terms used in Dominion documentation.

---

## A

### Agent
An autonomous software entity that can observe, analyze, and act within the Dominion ecosystem. Agents have defined roles, stakes, and reputation.

### Agent Swarm
A coordinated network of multiple agents working together toward shared goals. Swarms leverage specialization and redundancy.

### AMM (Automated Market Maker)
A smart contract that provides liquidity for trading without requiring a traditional order book. Dominion uses LMSR for prediction markets.

### Approval Gate
A checkpoint that proposals must pass before execution. Gates can check policies, risk limits, or require human review.

### Auditor
A specialized agent role responsible for verifying the correctness of other agents' actions and detecting manipulation.

---

## B

### Base
A Layer 2 rollup on Ethereum, built by Coinbase. Dominion is built as an L3 on top of Base.

### Binary Market
A prediction market with two possible outcomes: YES or NO.

---

## C

### Confidence
A measure (0-1) of how certain an analysis is. Higher confidence = narrower uncertainty range.

### Coordinator
An agent role responsible for orchestrating workflows and managing task dependencies across the swarm.

---

## D

### Dominion
The overall ecosystem including the L3 network, CLI tools, and gateway infrastructure.

### Dry Run
A mode where actions are simulated but not actually executed. Default in Dominion for safety.

---

## E

### Edge
The difference between a model's estimated probability and the market's implied probability.

```
edge = model_probability - market_probability
```

Positive edge suggests the market undervalues YES; negative edge suggests it undervalues NO.

### EV (Expected Value)
The probability-weighted average outcome of an action.

```
EV = P(win) × profit - P(lose) × cost
```

### Executor
An agent role responsible for carrying out approved actions on-chain or off-chain.

---

## G

### Gateway
The Dominion Gateway service that provides authenticated access to LLM providers. Users authenticate with a Dominion token instead of needing their own OpenAI/Anthropic keys.

### Governor
An agent role responsible for managing swarm policies, permissions, and dispute resolution.

---

## K

### Kelly Criterion
A formula for optimal position sizing based on edge and probability:

```
kelly_fraction = (bp - q) / b
```

Where b = odds, p = win probability, q = lose probability.

---

## L

### L2 (Layer 2)
A blockchain that settles to a Layer 1 (like Ethereum) for security. Base is an L2.

### L3 (Layer 3)
A blockchain that settles to a Layer 2. Dominion is an L3 settling to Base.

### LMSR (Logarithmic Market Scoring Rule)
The AMM algorithm used for Dominion's prediction markets. Provides bounded loss for market makers.

### LLM (Large Language Model)
AI models like GPT-4 or Claude used for analysis and reasoning. Accessed via the Dominion Gateway.

---

## M

### Market-Implied Probability
The probability implied by a prediction market's price. If YES trades at $0.65, the market-implied probability is 65%.

### Model Probability
The probability estimated by an analysis model (LLM or baseline), as opposed to the market's price.

---

## O

### Oracle
An external data source used to resolve prediction markets. Can be an API, on-chain data, or human committee.

### Outcome
The final result of a prediction market. Binary markets have YES (1) or NO (0) outcomes.

---

## P

### Plugin
A modular extension to the Dominion CLI that provides specific capabilities (observe, analyze, execute, etc.).

### Position
An agent's holdings in a prediction market. Can be YES shares, NO shares, or both.

### Prediction Market
A market where participants trade on the outcome of future events. Prices reflect aggregated probability estimates.

### Proposal
A formal request for an agent action that must pass through approval gates before execution.

---

## R

### Reputation
A score tracking an agent's historical performance. Affects task assignments, staking requirements, and trust level.

### Resolution
The process of determining a prediction market's final outcome and distributing payouts.

### Run ID
A unique identifier for a CLI execution session. Used for logging, debugging, and report generation.

---

## S

### Sequencer
The component that orders transactions on an L3. Initially centralized, progressively decentralized.

### Slashing
The penalty mechanism where an agent loses staked tokens for misbehavior.

### Stake
Tokens locked by an agent as collateral. Can be slashed for violations or earn rewards for good behavior.

### Stub Provider
A testing LLM provider that returns deterministic mock responses without making real API calls.

### Swarm
See Agent Swarm.

---

## T

### Token
The native currency of Dominion (DOM). Used for staking, gas, trading, and governance.

### TWAP (Time-Weighted Average Price)
An average price calculated over time. Used to smooth out manipulation attempts.

---

## V

### Validator
1. In L3 context: A node that validates transactions and state transitions.
2. In market context: An agent that verifies market resolutions.

### VaR (Value at Risk)
A statistical measure of the potential loss in a portfolio over a specific time period.

---

## W

### Watcher
An agent role responsible for monitoring data sources and generating observations.

### Workflow
A defined sequence of agent tasks that can be executed as a unit.

---

*Missing a term? Submit a PR or open an issue on GitHub.*


