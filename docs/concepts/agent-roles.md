# Agent Roles

Dominion defines six core agent roles. Each role has specific responsibilities, capabilities, and constraints.

## Role Overview

| Role | Primary Function | Key Capability |
|------|------------------|----------------|
| **Watcher** | Observe data sources | Real-time monitoring |
| **Analyst** | Process and reason | LLM inference |
| **Executor** | Take actions | Transaction submission |
| **Coordinator** | Orchestrate workflows | Task management |
| **Auditor** | Verify correctness | Cross-validation |
| **Governor** | Manage policies | Rule enforcement |

## Watcher Agents

**Purpose:** Monitor on-chain and off-chain data sources in real-time.

**Responsibilities:**
- Subscribe to blockchain events
- Poll external APIs
- Monitor social feeds
- Detect anomalies and triggers

**Outputs:**
- Structured observations
- Event notifications
- Data snapshots

**Example Use Cases:**
- Monitor token prices across DEXs
- Watch for governance proposals
- Track social sentiment signals

```yaml
# Watcher agent configuration
role: watcher
sources:
  - type: evm
    chain: base
    events: [Transfer, Swap]
  - type: api
    url: https://api.example.com/prices
    interval: 5s
```

## Analyst Agents

**Purpose:** Process observations and generate insights using reasoning capabilities.

**Responsibilities:**
- Interpret raw data
- Generate predictions
- Estimate probabilities
- Identify opportunities

**Outputs:**
- Structured analyses
- Probability estimates
- Confidence scores
- Recommendations

**Example Use Cases:**
- Analyze market conditions
- Estimate outcome probabilities
- Score opportunity quality

```yaml
# Analyst agent configuration
role: analyst
model: gpt-4o
temperature: 0.2
output_schema:
  probability: number
  confidence: number
  factors: string[]
```

## Executor Agents

**Purpose:** Execute approved actions on-chain or off-chain.

**Responsibilities:**
- Submit transactions
- Call external APIs
- Write files and reports
- Trigger webhooks

**Constraints:**
- Can only execute approved actions
- Subject to rate limits
- Must log all actions

**Example Use Cases:**
- Submit swap transactions
- Post to social media
- Generate reports

```yaml
# Executor agent configuration
role: executor
capabilities:
  - evm_transaction
  - webhook
  - file_write
approval_required: true
dry_run_default: true
```

## Coordinator Agents

**Purpose:** Orchestrate multi-agent workflows and manage task dependencies.

**Responsibilities:**
- Assign tasks to agents
- Manage execution order
- Handle failures and retries
- Aggregate results

**Outputs:**
- Task assignments
- Workflow status
- Completion reports

**Example Use Cases:**
- Orchestrate analysis pipelines
- Manage market-making loops
- Coordinate multi-step operations

## Auditor Agents

**Purpose:** Verify the correctness and honesty of other agents.

**Responsibilities:**
- Cross-validate analyses
- Check execution correctness
- Detect anomalies
- Report violations

**Special Properties:**
- Read access to all agent outputs
- Cannot be overridden by other agents
- Reports directly to governance

**Example Use Cases:**
- Verify prediction accuracy
- Audit transaction execution
- Detect manipulation attempts

## Governor Agents

**Purpose:** Manage swarm policies and enforce rules.

**Responsibilities:**
- Define approval policies
- Set rate limits
- Manage agent permissions
- Handle disputes

**Outputs:**
- Policy updates
- Permission grants/revocations
- Dispute resolutions

**Example Use Cases:**
- Update risk parameters
- Ban malicious agents
- Resolve market disputes

## Role Composition

Agents can hold multiple roles, but certain combinations are restricted:

| Combination | Allowed | Reason |
|-------------|---------|--------|
| Watcher + Analyst | Yes | Common pattern |
| Analyst + Executor | Restricted | Requires extra approval |
| Auditor + Executor | No | Conflict of interest |
| Governor + Any | Restricted | Power concentration |

## Custom Roles

Developers can define custom roles by extending base roles:

```yaml
# Custom role definition
role: custom_forecaster
extends: analyst
capabilities:
  - probability_estimation
  - market_trading
constraints:
  - max_position_size: 1000
  - required_confidence: 0.7
```

