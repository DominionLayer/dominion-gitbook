# Quickstart

Get running with Dominion in 5 minutes.

## 1. Install

```bash
pnpm add -g @dominion/polymarket-cli
```

## 2. Get a Token

Contact the Dominion team for a `DOMINION_API_TOKEN`.

## 3. Configure

```bash
export DOMINION_API_TOKEN=dom_your_token_here
dominion-pm whoami
```

## 4. Scan Markets

```bash
dominion-pm scan
```

## 5. Analyze

```bash
dominion-pm analyze <market_id>
```

## 6. Simulate

```bash
dominion-pm simulate <market_id> --position YES --size 100
```

## 7. Generate Report

```bash
dominion-pm report latest
```

## Next Steps

- [Configuration](configuration.md)
- [LLM Providers](llm-providers.md)
- [Testing](testing.md)
