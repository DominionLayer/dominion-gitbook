# LLM Providers

Dominion abstracts LLM access through a provider system. Users authenticate to the Dominion Gateway—**no OpenAI or Anthropic keys required**.

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        USER                                  │
│                          │                                   │
│              DOMINION_API_TOKEN                             │
│                          │                                   │
│                          ▼                                   │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                  DOMINION CLI                          │  │
│  │                       │                                │  │
│  │              GatewayProvider                           │  │
│  │                       │                                │  │
│  └───────────────────────│───────────────────────────────┘  │
└──────────────────────────│──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                  DOMINION GATEWAY                            │
│                                                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │    Auth     │  │    Rate     │  │   Usage     │         │
│  │   Check     │  │   Limit     │  │  Tracking   │         │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘         │
│         └────────────────┼────────────────┘                 │
│                          │                                   │
│                          ▼                                   │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                   LLM Router                           │  │
│  │                       │                                │  │
│  │         ┌─────────────┼─────────────┐                 │  │
│  │         ▼             ▼             ▼                 │  │
│  │   ┌─────────┐   ┌─────────┐   ┌─────────┐            │  │
│  │   │ OpenAI  │   │Anthropic│   │  Stub   │            │  │
│  │   │   API   │   │   API   │   │Provider │            │  │
│  │   └─────────┘   └─────────┘   └─────────┘            │  │
│  │         │             │                               │  │
│  │   OPENAI_API_KEY  ANTHROPIC_API_KEY                  │  │
│  │   (server-side)   (server-side)                      │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

## For Users

### Setup

Users only need one thing:

```bash
export DOMINION_API_TOKEN=dom_your_token_here
```

That's it. No OpenAI key, no Anthropic key.

### Get a Token

1. Contact Dominion team
2. Or self-host the gateway (see [Gateway Deployment](#self-hosting))

### Check Quota

```bash
dominion-pm whoami
```

Output:
```
User ID: abc123
Daily Requests: 50/1000 (950 remaining)
Daily Tokens: 25K/100K (75K remaining)
Monthly Spend: $2.50/$50.00
```

## Provider Types

### GatewayProvider (Default)

Routes through Dominion Gateway. Recommended for most users.

```yaml
# pm.config.yaml
gateway:
  url: "https://web-production-2fb66.up.railway.app"
  # Token via env var
```

### OpenAIProvider (Direct)

Direct connection to OpenAI. Requires your own API key.

```yaml
provider:
  default: "openai"
  openai:
    model: "gpt-4-turbo-preview"
    temperature: 0.2
```

```bash
export OPENAI_API_KEY=sk-your-key
```

### AnthropicProvider (Direct)

Direct connection to Anthropic. Requires your own API key.

```yaml
provider:
  default: "anthropic"
  anthropic:
    model: "claude-3-5-sonnet-20241022"
    temperature: 0.2
```

```bash
export ANTHROPIC_API_KEY=sk-ant-your-key
```

### StubProvider (Testing)

Returns deterministic responses. No API calls.

```yaml
provider:
  default: "stub"
```

## Provider Selection Logic

```
1. Is DOMINION_API_TOKEN set?
   └─ Yes → Use GatewayProvider
   └─ No → Continue

2. Is config.provider.default set?
   └─ Yes → Use that provider
   └─ No → Use StubProvider
```

## Available Models

### Via Gateway

| Provider | Models |
|----------|--------|
| OpenAI | gpt-4o, gpt-4o-mini, gpt-4-turbo, gpt-3.5-turbo |
| Anthropic | claude-3-5-sonnet, claude-3-opus, claude-3-haiku |

### Model Selection

The gateway auto-selects based on availability. Or specify:

```typescript
const response = await llm.complete({
  provider: 'openai',  // or 'anthropic' or 'auto'
  model: 'gpt-4o-mini',
  messages: [...],
});
```

## Rate Limits & Quotas

### Per-User Limits

| Limit | Default |
|-------|---------|
| Daily requests | 1,000 |
| Daily tokens | 100,000 |
| Monthly spend cap | $50 |
| Concurrent requests | 5 |

### When Exceeded

```json
{
  "error": "quota_exceeded",
  "message": "Daily request limit exceeded",
  "limit": 1000,
  "used": 1000,
  "resets_at": "2026-01-05T00:00:00Z"
}
```

## Cost Tracking

The gateway tracks costs per user:

| Model | Input (per 1K) | Output (per 1K) |
|-------|----------------|-----------------|
| gpt-4o | $0.005 | $0.015 |
| gpt-4o-mini | $0.00015 | $0.0006 |
| claude-3-5-sonnet | $0.003 | $0.015 |
| claude-3-haiku | $0.00025 | $0.00125 |

Check your usage:
```bash
dominion-pm whoami --json
```

## Self-Hosting the Gateway

For teams that want to run their own gateway:

### 1. Clone

```bash
git clone https://github.com/DominionLayer/dominion-server
cd dominion-server
```

### 2. Configure

```bash
cp env.example .env
# Edit .env:
# OPENAI_API_KEY=sk-your-key
# ANTHROPIC_API_KEY=sk-ant-your-key
# ADMIN_TOKEN=your-admin-token
```

### 3. Deploy

```bash
# Docker
docker-compose up -d

# Or Railway
railway up
```

### 4. Create Users

```bash
# Create user
curl -X POST http://localhost:3100/admin/users \
  -H "Authorization: Bearer YOUR_ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name": "User", "email": "user@example.com"}'

# Create API key
curl -X POST http://localhost:3100/admin/keys \
  -H "Authorization: Bearer YOUR_ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"user_id": "USER_ID"}'
```

### 5. Distribute Tokens

Give users their `dom_xxx` tokens. They set:

```bash
export DOMINION_API_URL=https://your-gateway.com
export DOMINION_API_TOKEN=dom_xxx
```

## Implementing a Custom Provider

```typescript
import { LLMProvider, LLMCompletionOptions, LLMResponse } from './base.js';

export class MyProvider extends LLMProvider {
  readonly name = 'my-provider';

  async complete(options: LLMCompletionOptions): Promise<LLMResponse> {
    // Call your LLM API
    const response = await fetch('https://my-llm-api.com/complete', {
      method: 'POST',
      body: JSON.stringify({
        messages: options.messages,
        temperature: options.temperature,
        max_tokens: options.maxTokens,
      }),
    });

    const data = await response.json();

    return {
      content: data.text,
      usage: {
        promptTokens: data.input_tokens,
        completionTokens: data.output_tokens,
        totalTokens: data.total_tokens,
      },
    };
  }
}
```

Register in `src/core/providers/index.ts`.

