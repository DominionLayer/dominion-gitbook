# Coordination vs Execution

A core design principle in Dominion: **coordination is harder than execution**.

## The Insight

Most autonomous agent failures come not from inability to execute, but from executing the wrong thing. The hard problem is:

- **What** should be done?
- **When** should it be done?
- **Who** should do it?
- **How** do we know it was done correctly?

Execution is comparatively simple once these questions are answered.

## Separation of Concerns

```
┌─────────────────────────────────────────────────────────────┐
│                   COORDINATION LAYER                         │
│                                                              │
│  ┌───────────┐    ┌───────────┐    ┌───────────┐           │
│  │ Observe   │ -> │ Analyze   │ -> │ Propose   │           │
│  └───────────┘    └───────────┘    └───────────┘           │
│                                          │                   │
│                                          ▼                   │
│                                    ┌───────────┐            │
│                                    │  Approve  │            │
│                                    └───────────┘            │
└─────────────────────────────────────────────────────────────┘
                                          │
                                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    EXECUTION LAYER                           │
│                                                              │
│                    ┌───────────┐                            │
│                    │  Execute  │                            │
│                    └───────────┘                            │
│                          │                                   │
│                          ▼                                   │
│                    ┌───────────┐                            │
│                    │  Verify   │                            │
│                    └───────────┘                            │
└─────────────────────────────────────────────────────────────┘
```

## Coordination Layer

The coordination layer is where intelligence happens:

### Observation
- What data is relevant?
- What sources are trustworthy?
- What signals matter?

### Analysis
- What does the data mean?
- What are the probabilities?
- What are the risks?

### Proposal
- What action should be taken?
- What are the expected outcomes?
- What could go wrong?

### Approval
- Is this action permitted?
- Does it pass safety checks?
- Has it been reviewed?

## Execution Layer

The execution layer is deliberately simple:

### Execute
- Submit the transaction
- Call the API
- Write the file

### Verify
- Did it succeed?
- Was the outcome correct?
- Should we retry?

## Why This Matters

### 1. Safety
Separating coordination from execution creates natural checkpoints where humans (or auditor agents) can intervene.

### 2. Auditability
Every decision has a clear trace: observation → analysis → proposal → approval → execution.

### 3. Modularity
Coordination logic can be updated without changing execution infrastructure.

### 4. Composability
Different coordination strategies can share the same execution layer.

## Approval Gates

The bridge between coordination and execution is the **approval gate**:

```
Proposal ──► Approval Gate ──► Execution
                  │
                  │ Checks:
                  │ - Policy compliance
                  │ - Risk limits
                  │ - Human review (optional)
                  │ - Auditor sign-off
                  │
                  ▼
            Pass/Reject
```

See [Approval Gates](../security/approval-gates.md) for implementation details.

## Anti-Pattern: Direct Execution

**Don't do this:**

```
Observation ──► Execute
```

This bypasses all safety mechanisms. Even "simple" actions should go through the coordination layer.

**Do this instead:**

```
Observation ──► Analyze ──► Propose ──► Approve ──► Execute
```

Even if approval is automatic, the structure enables auditing and future safety improvements.

