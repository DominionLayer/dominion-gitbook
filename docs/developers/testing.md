# Testing

Dominion uses Vitest for testing with comprehensive unit and integration tests.

## Test Structure

```
tests/
├── unit/
│   ├── config.test.ts
│   ├── estimators.test.ts
│   ├── edge.test.ts
│   ├── simulation.test.ts
│   └── provider.test.ts
└── integration/
    ├── full-workflow.test.ts
    └── gateway.test.ts
```

## Running Tests

```bash
# Run all tests
pnpm test

# Run with watch mode
pnpm test:watch

# Run specific file
pnpm test tests/unit/estimators.test.ts

# Run with coverage
pnpm test --coverage
```

## Unit Tests

### Testing Estimators

```typescript
import { describe, it, expect } from 'vitest';
import { BaselineEstimator } from '../../src/analysis/estimators/baseline-estimator.js';

describe('BaselineEstimator', () => {
  const estimator = new BaselineEstimator();

  it('should have correct metadata', () => {
    expect(estimator.name).toBe('baseline');
    expect(estimator.type).toBe('baseline');
  });

  it('should estimate probability', async () => {
    const market = {
      id: 'test_market',
      question: 'Test question?',
      yes_price: 0.65,
      no_price: 0.35,
      volume_24h: 10000,
      liquidity: 50000,
      end_date: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000).toISOString(),
    };

    const result = await estimator.estimate(market);

    expect(result.estimatedProbability).toBeGreaterThan(0);
    expect(result.estimatedProbability).toBeLessThan(1);
    expect(result.confidence).toBeGreaterThan(0);
    expect(result.keyFactors).toBeInstanceOf(Array);
  });
});
```

### Testing Edge Calculations

```typescript
import { describe, it, expect } from 'vitest';
import { calculateEdge, calculateEV } from '../../src/analysis/edge.js';

describe('Edge Calculations', () => {
  describe('calculateEdge', () => {
    it('should calculate positive edge', () => {
      const edge = calculateEdge(0.70, 0.60);
      expect(edge).toBeCloseTo(0.10, 2);
    });

    it('should calculate negative edge', () => {
      const edge = calculateEdge(0.50, 0.60);
      expect(edge).toBeCloseTo(-0.10, 2);
    });
  });

  describe('calculateEV', () => {
    it('should calculate expected value for YES', () => {
      const ev = calculateEV({
        probability: 0.70,
        entryPrice: 0.60,
        position: 'YES',
        size: 10,
      });
      
      expect(ev.expectedValue).toBeGreaterThan(0);
    });
  });
});
```

### Testing Configuration

```typescript
import { describe, it, expect, beforeEach, afterEach } from 'vitest';
import { loadConfig } from '../../src/core/config/loader.js';
import * as fs from 'fs';

describe('Configuration', () => {
  const testConfigPath = './test-config.yaml';

  beforeEach(() => {
    fs.writeFileSync(testConfigPath, `
      general:
        name: "Test"
      provider:
        default: "stub"
    `);
  });

  afterEach(() => {
    if (fs.existsSync(testConfigPath)) {
      fs.unlinkSync(testConfigPath);
    }
  });

  it('should load valid config', () => {
    const config = loadConfig(testConfigPath);
    expect(config.general.name).toBe('Test');
    expect(config.provider.default).toBe('stub');
  });

  it('should use defaults for missing values', () => {
    const config = loadConfig(testConfigPath);
    expect(config.scoring.min_liquidity).toBe(1000);
  });
});
```

## Integration Tests

### Full Workflow Test

```typescript
import { describe, it, expect, beforeAll, afterAll } from 'vitest';
import { initDatabase } from '../../src/core/db/database.js';
import { StubProvider } from '../../src/core/providers/stub.js';
import { BaselineEstimator } from '../../src/analysis/estimators/baseline-estimator.js';
import { simulate } from '../../src/simulation/simulator.js';

describe('Full Workflow Integration', () => {
  let db: any;

  beforeAll(async () => {
    db = await initDatabase(':memory:');
  });

  afterAll(async () => {
    await db.close();
  });

  it('should complete full workflow', async () => {
    // 1. Mock market data
    const market = {
      id: 'integration_test',
      question: 'Integration test?',
      yes_price: 0.55,
      no_price: 0.45,
      volume_24h: 5000,
      liquidity: 20000,
    };

    // 2. Run estimation
    const estimator = new BaselineEstimator();
    const analysis = await estimator.estimate(market);

    expect(analysis.estimatedProbability).toBeDefined();
    expect(analysis.confidence).toBeDefined();

    // 3. Run simulation
    const simResult = simulate({
      market,
      analysis,
      position: 'YES',
      size: 10,
      entryPrice: 0.55,
    });

    expect(simResult.expectedValue).toBeDefined();
    expect(simResult.scenarios).toHaveLength(3);

    // 4. Verify no real API calls were made
    // (StubProvider returns mock data)
  });
});
```

### Gateway Integration Test

```typescript
import { describe, it, expect, vi } from 'vitest';

// Mock undici
vi.mock('undici', () => ({
  request: vi.fn().mockImplementation(async (url) => {
    if (url.includes('/health')) {
      return {
        statusCode: 200,
        body: { json: async () => ({ status: 'ok' }) },
      };
    }
    if (url.includes('/v1/llm/complete')) {
      return {
        statusCode: 200,
        body: {
          json: async () => ({
            content: '{"estimated_probability": 0.65}',
            usage: { input_tokens: 100, output_tokens: 50 },
          }),
        },
      };
    }
  }),
}));

describe('Gateway Integration', () => {
  it('should complete LLM request through gateway', async () => {
    const { GatewayProvider } = await import('../../src/core/providers/gateway.js');
    
    const provider = new GatewayProvider(
      'http://localhost:3100',
      'dom_test_token'
    );

    const response = await provider.complete({
      messages: [{ role: 'user', content: 'Test' }],
    });

    expect(response.content).toBeDefined();
    expect(response.usage).toBeDefined();
  });
});
```

## Mock Providers

### StubProvider

For testing without real API calls:

```typescript
import { StubProvider } from '../src/core/providers/stub.js';

const provider = new StubProvider();

const response = await provider.complete({
  messages: [{ role: 'user', content: 'Test' }],
});

// Returns deterministic mock response
expect(response.content).toContain('estimated_probability');
```

### MockPolymarketClient

For testing without real Polymarket API:

```typescript
import { createMockClient } from '../src/polymarket/client.js';

const client = createMockClient();

const markets = await client.getActiveMarkets();
expect(markets.length).toBeGreaterThan(0);
```

## Test Utilities

### Creating Test Context

```typescript
// tests/helpers.ts

export async function createTestContext() {
  const db = await initDatabase(':memory:');
  const config = loadConfig('./test-config.yaml');
  const llm = new StubProvider();

  return { db, config, llm };
}

export function createMockMarket(overrides = {}) {
  return {
    id: 'mock_market',
    question: 'Mock question?',
    yes_price: 0.50,
    no_price: 0.50,
    volume_24h: 10000,
    liquidity: 50000,
    end_date: new Date(Date.now() + 30 * 24 * 60 * 60 * 1000).toISOString(),
    ...overrides,
  };
}
```

## Test Configuration

### vitest.config.ts

```typescript
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
    include: ['tests/**/*.test.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: ['node_modules', 'dist', 'tests'],
    },
    testTimeout: 10000,
  },
});
```

## CI/CD

### GitHub Actions

```yaml
# .github/workflows/test.yml

name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: pnpm/action-setup@v2
        with:
          version: 8
      
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'
      
      - run: pnpm install
      - run: pnpm build
      - run: pnpm test
```

## Best Practices

1. **Test in isolation**: Mock external dependencies
2. **Use meaningful assertions**: Not just `toBeDefined()`
3. **Test edge cases**: Empty inputs, invalid data
4. **Keep tests fast**: Use in-memory DB, mock APIs
5. **Name tests clearly**: Describe expected behavior


