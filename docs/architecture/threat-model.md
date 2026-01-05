# Threat Model

This document describes the security threats Dominion is designed to handle and the mitigations in place.

## Threat Categories

1. **Adversarial Agents**: Malicious agents within the swarm
2. **Market Manipulation**: Attacks on prediction markets
3. **Oracle Attacks**: Compromise of data sources
4. **Infrastructure Attacks**: Attacks on off-chain components
5. **Governance Attacks**: Abuse of governance mechanisms

---

## 1. Adversarial Agents

### Threat: Sybil Attacks
An attacker creates many agents to gain disproportionate influence.

**Mitigations:**
- Stake requirements for agent registration
- Reputation systems weight historical performance
- Rate limits per agent
- Correlation detection across agents

### Threat: False Data Injection
Watcher agents submit false observations.

**Mitigations:**
- Cross-validation with multiple watchers
- Stake slashing for detected lies
- Reputation impact for inaccurate data
- Source verification requirements

### Threat: Manipulative Analysis
Analyst agents provide biased or false analysis.

**Mitigations:**
- Prediction market incentives align with accuracy
- Auditor agents cross-check analyses
- Historical accuracy tracking
- Multiple analyst requirement for critical decisions

### Threat: Unauthorized Execution
Executor agents act without approval.

**Mitigations:**
- Cryptographic approval requirements
- On-chain verification of approvals
- Immediate slashing for violations
- Auditor monitoring of executions

---

## 2. Market Manipulation

### Threat: Price Manipulation
Attacker trades to move market price temporarily.

**Mitigations:**
- Position limits per agent
- Time-weighted average prices (TWAP)
- Large trade delays
- Manipulation detection algorithms

### Threat: Wash Trading
Attacker trades with themselves to create false volume.

**Mitigations:**
- Transaction fees make wash trading costly
- Volume-weighted metrics discount suspicious patterns
- Identity correlation detection

### Threat: Front-running
Attacker observes pending trades and trades ahead.

**Mitigations:**
- Commit-reveal schemes for large trades
- Encrypted order flow (planned)
- Fair ordering in sequencer

### Threat: Resolution Manipulation
Attacker influences how markets resolve.

**Mitigations:**
- Decentralized oracle networks
- Dispute periods before final resolution
- Multiple independent resolution sources
- Stake-weighted dispute voting

---

## 3. Oracle Attacks

### Threat: Data Source Compromise
External API provides false data.

**Mitigations:**
- Multiple independent data sources
- Outlier detection and filtering
- Source reputation tracking
- Manual override capability

### Threat: Stale Data
Data source provides outdated information.

**Mitigations:**
- Timestamp validation
- Freshness requirements
- Automatic source failover
- Heartbeat monitoring

### Threat: API Unavailability
Data sources become unavailable.

**Mitigations:**
- Redundant sources
- Graceful degradation
- Cached fallbacks (with warnings)
- Circuit breakers

---

## 4. Infrastructure Attacks

### Threat: Gateway Compromise
LLM Gateway is compromised.

**Mitigations:**
- Gateway doesn't hold user funds
- Rate limiting prevents abuse
- Audit logging for forensics
- Key rotation capabilities

### Threat: Database Corruption
Off-chain database is corrupted or lost.

**Mitigations:**
- Regular backups
- On-chain anchoring of critical data
- Merkle proofs for verification
- Disaster recovery procedures

### Threat: Sequencer Downtime
L3 sequencer becomes unavailable.

**Mitigations:**
- L2 escape hatch for withdrawals
- Sequencer rotation (planned)
- State commitments to L2

### Threat: DDoS
Services are overwhelmed with requests.

**Mitigations:**
- Rate limiting at all layers
- CDN and load balancing
- Stake-weighted priority queues
- Auto-scaling infrastructure

---

## 5. Governance Attacks

### Threat: Governance Capture
Attacker gains control of governance.

**Mitigations:**
- Time locks on governance actions
- Multi-sig requirements
- Veto mechanisms
- Progressive decentralization

### Threat: Parameter Manipulation
Malicious parameter changes (e.g., slashing = 100%).

**Mitigations:**
- Parameter bounds enforced in contracts
- Time locks for parameter changes
- Community monitoring and alerts

---

## Security Assumptions

> **Assumption:** The majority of staked value is held by honest actors.

> **Assumption:** At least one data source per category is honest.

> **Assumption:** LLM providers (OpenAI, Anthropic) do not collude with attackers.

> **Assumption:** Base L2 and Ethereum L1 remain secure.

---

## Incident Response

### Severity Levels

| Level | Description | Response Time |
|-------|-------------|---------------|
| P0 | Fund loss possible | Immediate |
| P1 | Service disruption | 1 hour |
| P2 | Degraded service | 4 hours |
| P3 | Minor issue | 24 hours |

### Response Procedures

1. **Detection**: Automated monitoring or user report
2. **Triage**: Assess severity and impact
3. **Containment**: Pause affected components if needed
4. **Investigation**: Determine root cause
5. **Resolution**: Fix and verify
6. **Post-mortem**: Document and improve

---

## Bug Bounty

Dominion operates a bug bounty program for responsible disclosure:

| Severity | Bounty Range |
|----------|--------------|
| Critical | $10,000 - $100,000 |
| High | $5,000 - $10,000 |
| Medium | $1,000 - $5,000 |
| Low | $100 - $1,000 |

Contact: security@dominion.network

