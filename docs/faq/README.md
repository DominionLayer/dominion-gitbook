# Frequently Asked Questions

## General

### Is Dominion an "AI chain"?

No. Dominion is a Layer 3 network optimized for **agent coordination**, not AI computation.

- AI inference happens off-chain (via LLM providers)
- The chain handles coordination, markets, and settlement
- Think of it as infrastructure *for* AI agents, not AI *on* chain

### Why L3 instead of L2?

L3 gives us:
- **Higher throughput** — Agents need 10,000+ TPS
- **Lower latency** — 100ms blocks vs 2s on L2
- **Cheaper gas** — 100x cheaper than L2
- **Custom primitives** — Native prediction markets

We still inherit Base L2's security through state commitments.

### Is Dominion decentralized?

Dominion follows **progressive decentralization**:

**Phase 1 (Current):**
- Centralized sequencer
- Decentralized validation

**Phase 2:**
- Shared sequencer with other L3s

**Phase 3:**
- Fully decentralized sequencer set

This is the same path taken by most successful L2s.

## Agents & Trading

### Can agents trade automatically?

**No, not by default.**

- All actions require explicit approval
- Dry-run mode is the default
- High-value actions require human review
- The `--approve` flag must be explicitly passed

Dominion is designed for analysis and coordination, not autonomous trading.

### Does the CLI provide financial advice?

**No.**

Every output includes disclaimers:
- "This is analysis only, not financial advice"
- Confidence/uncertainty ranges
- No guaranteed profit language

The tools help you analyze—decisions are yours.

### Can I make guaranteed profits?

**No.**

- Markets are inherently uncertain
- Models can be wrong
- Past performance ≠ future results
- "Edge" is an estimate, not a guarantee

Never risk more than you can afford to lose.

## Technical

### How do users access LLM features without their own keys?

Through the **Dominion Gateway**:

1. Users get a `DOMINION_API_TOKEN`
2. CLI uses this token to call the gateway
3. Gateway uses its own OpenAI/Anthropic keys
4. Users never see provider keys

This simplifies setup and allows usage tracking/limits.

### How do you prevent market manipulation?

Multiple mechanisms:

| Mechanism | Purpose |
|-----------|---------|
| Position limits | Cap per-agent exposure |
| TWAP pricing | Smooth manipulation spikes |
| Auditor agents | Detect suspicious patterns |
| Stake slashing | Punish bad actors |
| Correlation limits | Prevent coordinated attacks |

### What happens if the gateway goes down?

- CLI commands that need LLM will fail gracefully
- Baseline estimators (no LLM) still work
- Historical data remains in local database
- Markets on L3 continue operating

### Can I run my own gateway?

Yes! See [LLM Providers - Self Hosting](../developers/llm-providers.md#self-hosting-the-gateway).

You'll need:
- Your own OpenAI/Anthropic API keys
- A server to host the gateway
- To distribute tokens to your users

## Prediction Markets

### How are markets resolved?

1. **End time reached** — Trading stops
2. **Oracle reports** — External data source
3. **Dispute window** — 24h for challenges
4. **Settlement** — Winners paid

For disputed markets, resolution can escalate to governance.

### What if the oracle is wrong?

Anyone can raise a dispute:
1. Post dispute bond
2. Provide evidence
3. Resolution market or DAO decides
4. Winner gets loser's bond

### Are prediction markets legal?

**It depends on jurisdiction.**

- Dominion doesn't provide legal advice
- Users are responsible for compliance
- Some jurisdictions restrict prediction markets
- Consult local regulations

## Security

### What if my API token is leaked?

1. **Immediately** request a new token
2. Old token will be revoked
3. Audit your recent usage via `whoami`
4. Report to security@dominion.network

### Is my prompt data logged?

**No, by default.**

- Only metadata is logged (token counts, latency)
- Prompt content is redacted
- See [Privacy & Logging](../security/privacy.md)

### How do I report a security issue?

Email: security@dominion.network

- Use responsible disclosure
- Don't exploit vulnerabilities
- Bug bounties available for valid reports

## Getting Help

### Where can I get support?

- **Docs:** You're here!
- **Discord:** [discord.gg/dominion](https://discord.gg/dominion)
- **GitHub:** [github.com/DominionLayer](https://github.com/DominionLayer)
- **Email:** support@dominion.network

### How do I report a bug?

1. Check existing issues on GitHub
2. If new, create issue with:
   - Steps to reproduce
   - Expected behavior
   - Actual behavior
   - Run ID (if applicable)
   - Environment details

### Can I contribute?

Yes! See contribution guidelines in each repo:
- [dominion-swarm](https://github.com/DominionLayer/dominion-swarm)
- [prediction-layer](https://github.com/DominionLayer/prediction-layer)
- [dominion-server](https://github.com/DominionLayer/dominion-server)

