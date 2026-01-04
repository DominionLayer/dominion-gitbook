# Key Management

Dominion uses a centralized gateway model where users don't need their own LLM API keys.

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                         USERS                                │
│                                                              │
│  User A          User B          User C                     │
│     │               │               │                        │
│  dom_aaa        dom_bbb         dom_ccc                     │
│     │               │               │                        │
│     └───────────────┼───────────────┘                        │
│                     │                                        │
│                     ▼                                        │
└─────────────────────────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                  DOMINION GATEWAY                            │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │                  KEY STORAGE                          │   │
│  │                                                       │   │
│  │  User Keys (hashed):                                 │   │
│  │    hash(dom_aaa) → user_a_id                        │   │
│  │    hash(dom_bbb) → user_b_id                        │   │
│  │    hash(dom_ccc) → user_c_id                        │   │
│  │                                                       │   │
│  │  Provider Keys (encrypted at rest):                  │   │
│  │    OPENAI_API_KEY=sk-xxx                            │   │
│  │    ANTHROPIC_API_KEY=sk-ant-xxx                     │   │
│  │                                                       │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## User API Keys

### Format

```
dom_[random_32_chars]

Example: dom_ruAImsd3gkOt6xtrgb5H0D1g-2E7k47G
```

### Generation

```typescript
import { nanoid } from 'nanoid';
import * as argon2 from 'argon2';

async function generateApiKey(userId: string): Promise<{
  plaintext: string;
  hash: string;
}> {
  // Generate random key
  const keyBody = nanoid(32);
  const plaintext = `dom_${keyBody}`;
  
  // Hash for storage
  const hash = await argon2.hash(plaintext);
  
  return { plaintext, hash };
}
```

### Storage

Keys are **never stored in plaintext**:

```sql
CREATE TABLE api_keys (
  id TEXT PRIMARY KEY,
  user_id TEXT NOT NULL,
  key_hash TEXT NOT NULL,      -- argon2 hash
  key_prefix TEXT NOT NULL,    -- "dom_abc123" for lookup
  name TEXT,
  created_at TIMESTAMP,
  last_used_at TIMESTAMP,
  revoked_at TIMESTAMP
);
```

### Verification

```typescript
async function verifyApiKey(providedKey: string): Promise<User | null> {
  // Extract prefix for lookup
  const prefix = providedKey.slice(0, 12);
  
  // Find candidate keys
  const candidates = await db.query(
    'SELECT * FROM api_keys WHERE key_prefix = ? AND revoked_at IS NULL',
    [prefix]
  );
  
  // Verify against hash
  for (const candidate of candidates) {
    if (await argon2.verify(candidate.key_hash, providedKey)) {
      return await getUser(candidate.user_id);
    }
  }
  
  return null;
}
```

## Provider Keys

### Security Measures

Provider API keys (OpenAI, Anthropic) are:

1. **Never exposed to users** — Only gateway has access
2. **Encrypted at rest** — Using platform encryption
3. **Rotated regularly** — Monthly rotation
4. **Monitored** — Alerts on unusual usage

### Environment Variables

```bash
# Server-side only
OPENAI_API_KEY=sk-proj-xxxxx
ANTHROPIC_API_KEY=sk-ant-api03-xxxxx

# Encrypted in production
# - AWS: Secrets Manager
# - Railway: Encrypted variables
# - Docker: Docker secrets
```

## Key Lifecycle

### Creation

```
1. Admin creates user
2. Admin generates API key for user
3. Key displayed ONCE to admin
4. Admin securely delivers key to user
5. User stores key securely
```

### Usage

```
1. User includes key in request header
2. Gateway extracts and verifies key
3. Request processed if valid
4. Usage recorded against user
```

### Rotation

```
1. User requests new key (or admin creates)
2. New key generated
3. Old key marked for deprecation
4. Grace period (24h) for migration
5. Old key revoked
```

### Revocation

```typescript
await revokeApiKey({
  keyId: 'key_123',
  reason: 'User requested',
  immediate: false,  // Grace period
});
```

## Security Best Practices

### For Users

1. **Never share your key** — It's like a password
2. **Don't commit to git** — Use environment variables
3. **Rotate if exposed** — Request new key immediately
4. **Use environment variables** — Not hardcoded
5. **Monitor usage** — Check `whoami` regularly

### For Operators

1. **Hash all keys** — Never store plaintext
2. **Encrypt provider keys** — At rest and in transit
3. **Audit access** — Log all key operations
4. **Rate limit** — Prevent brute force
5. **Alert on anomalies** — Unusual usage patterns

## Revoking Compromised Keys

### Immediate Revocation

```bash
# Admin action
curl -X DELETE https://gateway/admin/keys/key_123 \
  -H "Authorization: Bearer ADMIN_TOKEN" \
  -d '{"immediate": true}'
```

### User Self-Service

```bash
# Future feature
dominion-pm keys revoke --all
dominion-pm keys rotate
```

## Audit Logging

All key operations are logged:

```json
{
  "event": "api_key_created",
  "timestamp": "2026-01-04T12:00:00Z",
  "user_id": "user_123",
  "key_id": "key_456",
  "created_by": "admin_789",
  "ip_address": "[REDACTED]"
}
```

```json
{
  "event": "api_key_used",
  "timestamp": "2026-01-04T12:01:00Z",
  "user_id": "user_123",
  "key_id": "key_456",
  "endpoint": "/v1/llm/complete",
  "status": "success"
}
```

## Recovery

### Lost Key

1. User contacts admin
2. Admin verifies identity
3. Old key revoked
4. New key generated
5. New key delivered securely

### Compromised Key

1. Immediately revoke key
2. Audit recent usage
3. Generate new key
4. Notify user
5. Investigate breach


