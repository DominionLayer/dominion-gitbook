# Plugin System

The Dominion CLI uses a modular plugin architecture.

## Built-in Plugins

| Plugin | Purpose |
|--------|---------|
| `observe` | Monitor data sources |
| `analyze` | Process with LLMs |
| `execute` | Carry out actions |
| `infra` | Infrastructure coordination |
| `market` | Market operations |
| `governance` | Policy management |

## Creating a Plugin

```typescript
import { Plugin, Tool } from '@dominion/core';

export class MyPlugin extends Plugin {
  name = 'my-plugin';
  
  tools: Tool[] = [
    {
      name: 'my_tool',
      description: 'Does something useful',
      execute: async (input) => {
        return result;
      },
    },
  ];
}
```

## Registering

```typescript
const cli = createCLI();
cli.use(new MyPlugin());
cli.run();
```
