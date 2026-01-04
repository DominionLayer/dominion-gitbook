# Auditor Agents

Auditor agents are specialized agents that verify the correctness and honesty of other agents.

## Role

Auditors serve as the "internal affairs" of an agent swarm:

- **Verify** — Check that actions match proposals
- **Validate** — Confirm data accuracy
- **Detect** — Find manipulation and fraud
- **Report** — Escalate violations

## Properties

### Independence

Auditors operate independently from other agents:

```
┌─────────────────────────────────────────────────────────────┐
│                    AGENT SWARM                               │
│                                                              │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                     │
│  │Watchers │  │Analysts │  │Executors│                     │
│  └────┬────┘  └────┬────┘  └────┬────┘                     │
│       │            │            │                            │
│       └────────────┼────────────┘                            │
│                    │                                         │
│                    ▼                                         │
│              ┌───────────┐                                   │
│              │  AUDITOR  │ ← Independent verification       │
│              │   AGENT   │                                   │
│              └─────┬─────┘                                   │
│                    │                                         │
│                    ▼                                         │
│              ┌───────────┐                                   │
│              │Governance │ ← Reports directly               │
│              └───────────┘                                   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Access Rights

| Access | Level |
|--------|-------|
| Read all agent outputs | Full |
| Read all proposals | Full |
| Read all executions | Full |
| Read agent configs | Limited |
| Modify other agents | None |
| Execute actions | Audit-only |

### Cannot Be Overridden

Auditor findings cannot be dismissed by other agents. Only governance can override.

## Responsibilities

### 1. Execution Verification

```typescript
class AuditorAgent {
  async verifyExecution(execution: Execution): Promise<AuditResult> {
    // Get the original proposal
    const proposal = await this.getProposal(execution.proposalId);
    
    // Check: Did execution match proposal?
    const matchesProposal = this.compareExecutionToProposal(
      execution, 
      proposal
    );
    
    // Check: Was result as expected?
    const resultCorrect = await this.verifyResult(execution);
    
    // Check: Were approvals valid?
    const approvalsValid = await this.verifyApprovals(
      execution.approvalReceipt
    );
    
    return {
      executionId: execution.id,
      checks: {
        matchesProposal,
        resultCorrect,
        approvalsValid,
      },
      passed: matchesProposal && resultCorrect && approvalsValid,
      timestamp: new Date().toISOString(),
    };
  }
}
```

### 2. Data Validation

```typescript
async validateData(observation: Observation): Promise<ValidationResult> {
  // Cross-check with independent sources
  const sources = await this.queryIndependentSources(
    observation.type,
    observation.timestamp
  );
  
  // Calculate deviation
  const deviation = this.calculateDeviation(
    observation.value,
    sources
  );
  
  // Flag if suspicious
  if (deviation > this.deviationThreshold) {
    return {
      valid: false,
      observation,
      deviation,
      sources,
      recommendation: 'INVESTIGATE',
    };
  }
  
  return { valid: true, observation };
}
```

### 3. Pattern Detection

```typescript
async detectPatterns(timeWindow: string): Promise<Alert[]> {
  const alerts: Alert[] = [];
  
  // Detect unusual trading patterns
  const tradingAlerts = await this.analyzeTrading(timeWindow);
  alerts.push(...tradingAlerts);
  
  // Detect collusion signals
  const collusionAlerts = await this.analyzeCollusion(timeWindow);
  alerts.push(...collusionAlerts);
  
  // Detect performance anomalies
  const performanceAlerts = await this.analyzePerformance(timeWindow);
  alerts.push(...performanceAlerts);
  
  return alerts;
}
```

### 4. Market Resolution Verification

```typescript
async verifyResolution(market: Market): Promise<ResolutionVerification> {
  // Get reported outcome
  const reported = market.resolution;
  
  // Query independent sources
  const sources = await this.getResolutionSources(market);
  
  // Determine consensus
  const consensus = this.calculateConsensus(sources);
  
  // Compare
  if (reported.outcome !== consensus.outcome) {
    await this.flagResolutionDiscrepancy({
      market,
      reported: reported.outcome,
      expected: consensus.outcome,
      sources,
    });
    
    return {
      verified: false,
      discrepancy: true,
      recommendation: 'DISPUTE',
    };
  }
  
  return { verified: true };
}
```

## Alert Types

| Alert | Severity | Auto-Response |
|-------|----------|---------------|
| Execution mismatch | Critical | Pause agent |
| Data deviation | High | Flag for review |
| Pattern anomaly | Medium | Increase monitoring |
| Resolution discrepancy | Critical | Initiate dispute |
| Performance degradation | Low | Log and report |

## Auditor Selection

Auditors are selected randomly to prevent gaming:

```typescript
function selectAuditors(proposal: Proposal): Auditor[] {
  // Get all eligible auditors
  const eligible = auditors.filter(a => 
    a.status === 'ACTIVE' &&
    a.reputation >= MIN_REPUTATION &&
    !a.hasConflict(proposal)
  );
  
  // Random selection weighted by reputation
  const selected = weightedRandomSample(
    eligible,
    a => a.reputation,
    AUDITORS_PER_PROPOSAL
  );
  
  return selected;
}
```

## Incentives

### Rewards

| Action | Reward |
|--------|--------|
| Correct verification | Base fee |
| Caught violation | Bounty (10% of stake) |
| First to report | Discovery bonus |

### Penalties

| Failure | Penalty |
|---------|---------|
| Missed violation | Stake slash (5%) |
| False accusation | Stake slash (10%) |
| Collusion | Full stake loss + ban |

## Configuration

```yaml
# Auditor configuration
auditor:
  # Minimum auditors per high-value proposal
  min_auditors: 2
  
  # Minimum reputation to audit
  min_reputation: 0.7
  
  # Verification timeout
  timeout_ms: 30000
  
  # Alert thresholds
  thresholds:
    data_deviation: 0.05
    performance_anomaly: 2.0  # std devs
    pattern_score: 0.8
```

## Becoming an Auditor

### Requirements

1. Minimum stake (higher than standard agents)
2. Clean history (no violations)
3. Technical capability (run auditor software)
4. Pass auditor exam (planned)

### Registration

```typescript
await registry.registerAuditor({
  agentId: 'auditor_001',
  stake: 10000,
  capabilities: ['execution', 'data', 'market'],
  availability: '24/7',
});
```


