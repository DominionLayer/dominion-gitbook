# Dominion GitBook

Documentation content for Dominion - the coordination layer for autonomous agent swarms on Base.

## Overview

This repository contains the source documentation in GitBook-compatible Markdown format. The content is synced to the [dominion-docs](https://github.com/DominionLayer/dominion-docs) website.

## Structure

```
dominion-gitbook/
├── SUMMARY.md           # Navigation structure
├── docs/
│   ├── introduction/    # Introduction section
│   ├── concepts/        # Core concepts
│   ├── architecture/    # Technical architecture
│   ├── products/        # Product documentation
│   ├── developers/      # Developer guides
│   ├── prediction-markets/  # Market documentation
│   ├── security/        # Security documentation
│   ├── faq/            # FAQ
│   └── glossary/       # Terminology
└── assets/             # Images and diagrams
```

## Documentation Sections

| Section | Description |
|---------|-------------|
| Introduction | What is Dominion, why L3 |
| Concepts | Agent swarms, coordination, incentives |
| Architecture | System design, data flow, threat model |
| Products | Network, CLIs, gateway |
| Developer Docs | Quickstart, config, plugins, testing |
| Prediction Markets | Lifecycle, participation, safety |
| Security | Gates, auditors, keys, privacy |
| FAQ | Common questions |
| Glossary | Term definitions |

## Usage

### With GitBook

1. Connect this repo to GitBook
2. Configure `SUMMARY.md` as the table of contents
3. Publish

### With dominion-docs Website

The website pulls content from this repo:

```bash
# In dominion-docs repo
pnpm sync
```

## Contributing

1. Fork the repo
2. Edit Markdown files
3. Submit PR

### Guidelines

- Keep language technical and clear
- Include disclaimers where appropriate
- No hype or marketing speak
- Use consistent terminology (see Glossary)

## Disclaimers

This documentation describes experimental software. All market-related content includes:

- "This is analysis only, not financial advice"
- Confidence/uncertainty ranges
- Risk warnings

## License

MIT
