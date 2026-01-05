# Rate Limiting

Rate limiting prevents abuse and ensures fair resource allocation.

## Rate Limit Types

### 1. API Rate Limits

Limits on gateway API calls:

| Endpoint | Limit | Window |
|----------|-------|--------|
| `/v1/llm/complete` | 60 req | 1 min |
| `/v1/llm/models` | 100 req | 1 min |
| `/admin/*` | 30 req | 1 min |

### 2. Agent Action Limits

Limits on agent actions:

| Action | Limit | Window |
|--------|-------|--------|
| Trades | 10 | 1 min |
| Proposals | 5 | 1 min |
| Observations | 100 | 1 min |
| Analyses | 20 | 1 min |

### 3. Resource Quotas

Daily/monthly resource limits:

| Resource | Daily | Monthly |
|----------|-------|---------|
| LLM requests | 1,000 | 25,000 |
| LLM tokens | 100,000 | 2,500,000 |
| API calls | 10,000 | 250,000 |
| Spend (USD) | $10 | $50 |

## Implementation

### Token Bucket Algorithm

```typescript
class TokenBucket {
  private tokens: number;
  private lastRefill: number;
  
  constructor(
    private capacity: number,
    private refillRate: number, // tokens per second
  ) {
    this.tokens = capacity;
    this.lastRefill = Date.now();
  }
  
  async consume(count: number = 1): Promise<boolean> {
    this.refill();
    
    if (this.tokens >= count) {
      this.tokens -= count;
      return true;
    }
    
    return false;
  }
  
  private refill(): void {
    const now = Date.now();
    const elapsed = (now - this.lastRefill) / 1000;
    const newTokens = elapsed * this.refillRate;
    
    this.tokens = Math.min(this.capacity, this.tokens + newTokens);
    this.lastRefill = now;
  }
  
  getWaitTime(count: number = 1): number {
    this.refill();
    
    if (this.tokens >= count) {
      return 0;
    }
    
    const needed = count - this.tokens;
    return (needed / this.refillRate) * 1000;
  }
}
```

### Rate Limiter Middleware

```typescript
import { RateLimiter } from './rate-limiter.js';

const limiter = new RateLimiter({
  windowMs: 60 * 1000, // 1 minute
  max: 60,             // 60 requests per window
});

// In route handler
app.post('/v1/llm/complete', async (req, res) => {
  const userId = req.user.id;
  
  const allowed = await limiter.check(userId);
  
  if (!allowed) {
    return res.status(429).json({
      error: 'rate_limit_exceeded',
      message: 'Too many requests',
      retry_after: limiter.getRetryAfter(userId),
    });
  }
  
  // Process request...
});
```

## Exponential Backoff

When rate limited, clients should use exponential backoff:

```typescript
async function withBackoff<T>(
  fn: () => Promise<T>,
  options: BackoffOptions = {}
): Promise<T> {
  const {
    maxRetries = 3,
    baseDelay = 1000,
    maxDelay = 30000,
  } = options;
  
  let attempt = 0;
  
  while (true) {
    try {
      return await fn();
    } catch (error) {
      if (!isRetryable(error) || attempt >= maxRetries) {
        throw error;
      }
      
      attempt++;
      const delay = Math.min(
        baseDelay * Math.pow(2, attempt) + Math.random() * 1000,
        maxDelay
      );
      
      await sleep(delay);
    }
  }
}

function isRetryable(error: unknown): boolean {
  if (error instanceof HTTPError) {
    return error.status === 429 || error.status >= 500;
  }
  return false;
}
```

## Quota Management

### Checking Quota

```bash
# CLI
dominion-pm whoami

# Output:
# Daily Requests: 500/1000 (500 remaining)
# Daily Tokens: 50K/100K (50K remaining)
```

### API

```typescript
// GET /v1/llm/quota
const response = await fetch('/v1/llm/quota', {
  headers: { 'Authorization': `Bearer ${token}` }
});

const quota = await response.json();
// {
//   daily_requests: { limit: 1000, used: 500, remaining: 500 },
//   daily_tokens: { limit: 100000, used: 50000, remaining: 50000 },
//   monthly_spend: { cap_usd: 50, used_usd: 12.50, remaining_usd: 37.50 }
// }
```

### Quota Exceeded Response

```json
{
  "error": "quota_exceeded",
  "message": "Daily token limit exceeded",
  "quota": {
    "type": "daily_tokens",
    "limit": 100000,
    "used": 100000,
    "remaining": 0,
    "resets_at": "2026-01-05T00:00:00Z"
  }
}
```

## Configuration

### Gateway Config

```yaml
# Rate limiting configuration
rate_limits:
  # Per-user limits
  per_user:
    requests_per_minute: 60
    requests_per_hour: 1000
    
  # Per-IP limits (unauthenticated)
  per_ip:
    requests_per_minute: 10
    requests_per_hour: 100
    
  # Global limits
  global:
    requests_per_second: 1000
```

### CLI Config

```yaml
# Client-side rate limiting
rate_limits:
  # Polymarket API
  polymarket_rps: 5
  
  # LLM calls
  llm_rpm: 20
  
  # Retry configuration
  max_retries: 3
  base_delay_ms: 1000
  max_delay_ms: 30000
```

## Best Practices

### For Developers

1. **Respect rate limits**: Don't circumvent them
2. **Use exponential backoff**: Don't hammer on 429
3. **Cache responses**: Reduce unnecessary calls
4. **Batch requests**: Combine where possible
5. **Monitor usage**: Track quota consumption

### For Operators

1. **Set appropriate limits**: Based on infrastructure capacity
2. **Monitor abuse**: Track suspicious patterns
3. **Adjust dynamically**: Increase/decrease based on load
4. **Communicate limits**: Document clearly for users


