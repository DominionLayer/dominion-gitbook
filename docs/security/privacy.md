# Privacy & Logging

What Dominion logs, what it doesn't, and how to control it.

## Privacy Principles

1. **Minimal Collection**: Only log what's necessary
2. **Prompt Redaction**: Prompt contents not logged by default
3. **User Control**: Users can request data deletion
4. **Transparency**: Clear documentation of what's logged

## What IS Logged

### Request Metadata

```json
{
  "request_id": "req_abc123",
  "timestamp": "2026-01-04T12:00:00.000Z",
  "user_id": "user_xyz",
  "endpoint": "/v1/llm/complete",
  "method": "POST",
  "status": 200,
  "latency_ms": 850,
  "ip_hash": "abc123..."  // Hashed, not raw IP
}
```

### LLM Usage (Metadata Only)

```json
{
  "request_id": "req_abc123",
  "provider": "openai",
  "model": "gpt-4o-mini",
  "input_tokens": 150,
  "output_tokens": 75,
  "total_tokens": 225,
  "cost_usd": 0.000135,
  "message_count": 2      // Not content
}
```

### Agent Actions

```json
{
  "action_id": "act_def456",
  "agent_id": "agent_001",
  "action_type": "analyze",
  "target": "market_xyz",
  "status": "completed",
  "duration_ms": 1200,
  "timestamp": "2026-01-04T12:00:00.000Z"
}
```

## What is NOT Logged

By default, these are **never logged**:

| Data | Logged? | Reason |
|------|---------|--------|
| Prompt content | No | Privacy |
| Response content | No | Privacy |
| Raw IP addresses | No | Hashed only |
| API key plaintext | No | Security |
| Personal info | No | GDPR/Privacy |

## Prompt Redaction

### Default Behavior

```typescript
function logLLMRequest(request: LLMRequest) {
  logger.info('LLM request', {
    provider: request.provider,
    model: request.model,
    messages: '[REDACTED]',      // Content redacted
    message_count: request.messages.length,
    temperature: request.temperature,
  });
}
```

### Full Logging (Dev Only)

For debugging, full logging can be enabled:

```yaml
# ONLY for development
logging:
  include_prompts: true  # NOT recommended for production
```

With this enabled:
```json
{
  "messages": [
    {"role": "system", "content": "You are..."},
    {"role": "user", "content": "Analyze..."}
  ]
}
```

> **Warning:** Never enable in production. Prompts may contain sensitive user data.

## Log Retention

| Log Type | Retention | Reason |
|----------|-----------|--------|
| Request logs | 30 days | Debugging |
| Usage metrics | 90 days | Billing |
| Audit logs | 1 year | Compliance |
| Error logs | 90 days | Debugging |
| Security logs | 2 years | Forensics |

## User Data Rights

### Data Export

Users can request their data:

```bash
# Future feature
dominion-pm data export --format json
```

Export includes:
- Account information
- Usage history (metadata)
- Billing records
- Audit logs (user's actions)

### Data Deletion

Users can request deletion:

```bash
# Future feature
dominion-pm data delete --confirm
```

Deletion removes:
- Account
- API keys
- Usage history
- Associated logs

Retention exceptions:
- Aggregated statistics (anonymized)
- Legal hold data
- Billing records (legal requirement)

## Anonymization

### IP Addresses

```typescript
function hashIP(ip: string): string {
  const salt = process.env.IP_HASH_SALT;
  return crypto.createHash('sha256')
    .update(ip + salt)
    .digest('hex')
    .slice(0, 16);
}
```

### User IDs in Logs

```typescript
function anonymizeForExternalLogs(userId: string): string {
  // For logs shared with third parties
  return crypto.createHash('sha256')
    .update(userId)
    .digest('hex')
    .slice(0, 8);
}
```

## Configuration

```yaml
# privacy.yaml

logging:
  # What to log
  log_requests: true
  log_responses: false
  log_prompts: false
  
  # Redaction
  redact:
    - field: "messages"
      replacement: "[REDACTED]"
    - field: "api_key"
      replacement: "[REDACTED]"
    - field: "email"
      replacement: "[EMAIL]"
  
  # Retention
  retention_days:
    requests: 30
    usage: 90
    audit: 365
    errors: 90

# IP handling
ip_handling:
  store_raw: false
  hash_algorithm: "sha256"
  hash_salt_env: "IP_HASH_SALT"
```

## Third-Party Services

### LLM Providers

When using OpenAI/Anthropic:
- Prompts are sent to provider
- Provider's privacy policy applies
- We don't control provider logging

### Analytics (Optional)

If analytics enabled:
- Only aggregated data
- No PII
- No prompt content

## Compliance

### GDPR

- Right to access: Supported
- Right to deletion: Supported
- Data portability: Supported
- Consent: Required for optional features

### SOC 2

- Audit logging: Yes
- Access controls: Yes
- Encryption: Yes
- Monitoring: Yes

## Security Incident Response

If logs are compromised:

1. **Immediate:** Revoke affected credentials
2. **24h:** Notify affected users
3. **72h:** File regulatory notifications
4. **Ongoing:** Forensic investigation

## Contact

Privacy concerns: privacy@dominion.network
Security issues: security@dominion.network


