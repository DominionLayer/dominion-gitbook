# Approval Gates

Approval gates are checkpoints that all agent actions must pass before execution.

## Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    APPROVAL PIPELINE                         │
│                                                              │
│  Proposal ──► Gate 1 ──► Gate 2 ──► Gate 3 ──► Execute     │
│                 │          │          │                      │
│                 ▼          ▼          ▼                      │
│              Policy      Risk      Human                     │
│              Check       Check     Review                    │
│                                                              │
│  Any gate can REJECT ──► Proposal blocked                   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## Gate Types

### 1. Policy Gate

Checks against defined rules and constraints.

```typescript
interface PolicyGate {
  name: 'policy';
  
  async check(proposal: Proposal): Promise<GateResult> {
    const checks = [
      this.checkAllowedActions(proposal),
      this.checkAllowedTargets(proposal),
      this.checkAllowedAgents(proposal),
      this.checkTimeRestrictions(proposal),
    ];
    
    const results = await Promise.all(checks);
    const failed = results.filter(r => !r.passed);
    
    return {
      passed: failed.length === 0,
      details: results,
    };
  }
}
```

**Policy Examples:**
- Only certain actions allowed
- Only certain markets tradeable
- No actions during maintenance windows
- Agent role must match action type

### 2. Risk Gate

Evaluates risk parameters.

```typescript
interface RiskGate {
  name: 'risk';
  
  async check(proposal: Proposal): Promise<GateResult> {
    const exposure = await this.calculateExposure(proposal);
    const var95 = await this.calculateVaR(proposal, 0.95);
    const correlation = await this.checkCorrelation(proposal);
    
    return {
      passed: (
        exposure <= this.maxExposure &&
        var95 <= this.maxVaR &&
        correlation <= this.maxCorrelation
      ),
      metrics: { exposure, var95, correlation },
    };
  }
}
```

**Risk Checks:**
- Position size limits
- Portfolio exposure limits
- Correlation limits
- Value at Risk (VaR)

### 3. Rate Gate

Prevents excessive activity.

```typescript
interface RateGate {
  name: 'rate';
  
  async check(proposal: Proposal): Promise<GateResult> {
    const recent = await this.getRecentActions(proposal.agentId);
    const rate = recent.length;
    
    return {
      passed: rate < this.maxActionsPerMinute,
      metrics: { current: rate, limit: this.maxActionsPerMinute },
    };
  }
}
```

### 4. Auditor Gate

Cross-validation by auditor agents.

```typescript
interface AuditorGate {
  name: 'auditor';
  
  async check(proposal: Proposal): Promise<GateResult> {
    // Get auditor opinions
    const auditors = await this.getAssignedAuditors(proposal);
    const opinions = await Promise.all(
      auditors.map(a => a.review(proposal))
    );
    
    // Require majority approval
    const approvals = opinions.filter(o => o.approved).length;
    const required = Math.ceil(auditors.length / 2);
    
    return {
      passed: approvals >= required,
      opinions,
    };
  }
}
```

### 5. Human Gate

Manual human review for high-stakes actions.

```typescript
interface HumanGate {
  name: 'human';
  
  async check(proposal: Proposal): Promise<GateResult> {
    // Check if human review required
    if (!this.requiresHumanReview(proposal)) {
      return { passed: true, skipped: true };
    }
    
    // Create review request
    const reviewId = await this.createReviewRequest(proposal);
    
    // Wait for human decision (with timeout)
    const decision = await this.waitForDecision(reviewId, {
      timeout: 24 * 60 * 60 * 1000, // 24 hours
    });
    
    return {
      passed: decision.approved,
      reviewer: decision.reviewer,
      notes: decision.notes,
    };
  }
  
  private requiresHumanReview(proposal: Proposal): boolean {
    return (
      proposal.value > this.humanReviewThreshold ||
      proposal.action.type === 'HIGH_RISK' ||
      proposal.flags.includes('MANUAL_REVIEW')
    );
  }
}
```

## Configuration

```yaml
# dominion.config.yaml

safety:
  approval_gates:
    - name: policy
      enabled: true
      config:
        allowed_actions: [trade, analyze, observe]
        blocked_markets: []
        
    - name: risk
      enabled: true
      config:
        max_position: 0.10
        max_exposure: 0.50
        max_var_95: 0.20
        
    - name: rate
      enabled: true
      config:
        max_per_minute: 10
        max_per_hour: 100
        
    - name: auditor
      enabled: true
      config:
        min_auditors: 2
        approval_threshold: 0.5
        
    - name: human
      enabled: true
      config:
        threshold_value: 10000
        timeout_hours: 24
```

## Dry Run Mode

By default, all actions run in dry-run mode:

```bash
# Dry run (default) - no actual execution
dominion run workflow

# Real execution - requires explicit flag
dominion run workflow --approve
```

In dry run:
- All gates are evaluated
- Results are logged
- No actual execution occurs
- Output shows "would execute"

```
DRY RUN: Trade proposal
  Market: eth_jan_2026
  Action: BUY YES
  Size: 100
  
Gate Results:
  [PASS] Policy: Action allowed
  [PASS] Risk: Exposure within limits
  [PASS] Rate: Under rate limit
  [SKIP] Human: Not required
  
Would execute: BUY 100 YES @ 0.62

To execute for real, run with --approve
```

## Approval Receipt

Every approved action gets a cryptographic receipt:

```typescript
interface ApprovalReceipt {
  proposalId: string;
  proposalHash: string;
  gates: {
    name: string;
    passed: boolean;
    timestamp: string;
    signature: string;
  }[];
  finalApproval: {
    timestamp: string;
    approver: string;
    signature: string;
  };
  validUntil: string;
}
```

Receipts are:
- Required for execution
- Time-limited (expire after 5 minutes)
- Logged for audit
- Verifiable on-chain

## Custom Gates

Developers can add custom gates:

```typescript
import { ApprovalGate, GateResult, Proposal } from '../types.js';

export class MyCustomGate implements ApprovalGate {
  name = 'my-custom-gate';
  
  async check(proposal: Proposal): Promise<GateResult> {
    // Custom logic
    const myCheck = await this.performCustomCheck(proposal);
    
    return {
      passed: myCheck.ok,
      details: myCheck.details,
    };
  }
  
  private async performCustomCheck(proposal: Proposal) {
    // Implementation
  }
}
```

Register in config:
```yaml
safety:
  approval_gates:
    - name: my-custom-gate
      enabled: true
      module: "./gates/my-custom-gate.js"
```

