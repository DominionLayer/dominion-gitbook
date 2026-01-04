# On-chain vs Off-chain

Dominion strategically splits responsibilities between on-chain and off-chain components.

## Design Philosophy

**On-chain:** Things that need trustlessness, finality, or economic enforcement.

**Off-chain:** Things that need speed, flexibility, or don't require trustlessness.

## Component Placement

| Component | Location | Rationale |
|-----------|----------|-----------|
| Agent Registry | On-chain | Trust: who is allowed to act |
| Staking/Slashing | On-chain | Economic enforcement |
| Market Settlement | On-chain | Trust: final outcomes |
| Governance | On-chain | Credible commitments |
| Observation | Off-chain | Speed: real-time data |
| Analysis | Off-chain | Flexibility: LLM calls |
| Coordination | Hybrid | Speed + audit trail |
| Approval Logic | Off-chain | Flexibility: policy updates |
| Execution | Hybrid | Depends on target |
| Logging | Off-chain | Volume: too much data |

## On-chain Components

### Agent Registry

```solidity
contract AgentRegistry {
    struct Agent {
        address owner;
        bytes32 role;
        uint256 stake;
        uint256 reputation;
        bool active;
    }
    
    mapping(bytes32 => Agent) public agents;
    
    function register(bytes32 agentId, bytes32 role) external payable;
    function deregister(bytes32 agentId) external;
    function slash(bytes32 agentId, uint256 amount) external;
}
```

### Prediction Markets

```solidity
contract PredictionMarket {
    struct Market {
        bytes32 id;
        string question;
        uint256 endTime;
        uint256 yesPool;
        uint256 noPool;
        bool resolved;
        bool outcome;
    }
    
    function createMarket(string memory question, uint256 endTime) external;
    function buy(bytes32 marketId, bool yes, uint256 amount) external;
    function resolve(bytes32 marketId, bool outcome) external;
}
```

### Staking

```solidity
contract Staking {
    mapping(bytes32 => uint256) public stakes;
    
    function stake(bytes32 agentId) external payable;
    function unstake(bytes32 agentId, uint256 amount) external;
    function slash(bytes32 agentId, uint256 amount) external onlyGovernance;
}
```

## Off-chain Components

### LLM Gateway

Handles LLM API calls with:
- Authentication
- Rate limiting
- Cost tracking
- Response caching

```typescript
interface LLMGateway {
  complete(request: CompletionRequest): Promise<CompletionResponse>;
  getUsage(userId: string): Promise<UsageStats>;
  getRemainingQuota(userId: string): Promise<Quota>;
}
```

### Approval Service

Evaluates proposals against policies:

```typescript
interface ApprovalService {
  evaluate(proposal: Proposal): Promise<ApprovalResult>;
  getPolicy(policyId: string): Promise<Policy>;
  updatePolicy(policyId: string, policy: Policy): Promise<void>;
}
```

### Audit Log

Stores all agent actions:

```typescript
interface AuditLog {
  record(event: AuditEvent): Promise<void>;
  query(filter: AuditFilter): Promise<AuditEvent[]>;
  getProof(eventId: string): Promise<MerkleProof>;
}
```

## Hybrid Components

### Coordination

- **Off-chain:** Real-time coordination messages
- **On-chain:** Commitment hashes for dispute resolution

### Execution

- **Off-chain targets:** API calls, file writes → pure off-chain
- **On-chain targets:** L3/L2 transactions → on-chain execution

## Data Anchoring

Off-chain data is anchored on-chain for verifiability:

```
┌─────────────────────────────────────────────────────────────┐
│                     OFF-CHAIN                                │
│                                                              │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐        │
│  │ Event 1 │  │ Event 2 │  │ Event 3 │  │ Event N │        │
│  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘        │
│       │            │            │            │               │
│       └────────────┴─────┬──────┴────────────┘               │
│                          │                                   │
│                   ┌──────▼──────┐                           │
│                   │ Merkle Root │                           │
│                   └──────┬──────┘                           │
└──────────────────────────│──────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────────┐
│                      ON-CHAIN (L3)                            │
│                                                               │
│                   ┌─────────────┐                            │
│                   │ Anchor Hash │                            │
│                   │  Contract   │                            │
│                   └─────────────┘                            │
└──────────────────────────────────────────────────────────────┘
```

This allows:
- Efficient off-chain storage
- On-chain verifiability
- Dispute resolution with proofs

## Trade-offs

| On-chain | Off-chain |
|----------|-----------|
| Trustless | Requires trust |
| Slow (~100ms) | Fast (~10ms) |
| Expensive | Cheap |
| Permanent | Ephemeral (by default) |
| Public | Private (optional) |
| Immutable | Updateable |

