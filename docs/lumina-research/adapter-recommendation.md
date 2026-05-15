# Adapter Recommendation

To integrate Dexter's capabilities into Lumina OS V3 without taking on the maintenance burden of its CLI or specific external gateways, the recommended approach is to build an **API Wrapper / Provider Abstraction**.

## Recommended Integration Shape: API Wrapper

Instead of running Dexter via `CLI`, Lumina should interact with a thin wrapper around `Agent.create()` and `runContext`.

### Proposed Minimal Interface

```typescript
// lumina-dexter-adapter.ts

import { Agent } from 'dexter-ts/src/agent/agent';
import { MemoryManager } from 'dexter-ts/src/memory/index';

export class DexterProvider {
  private agent: Agent;

  constructor(config: { model: string, memoryEnabled: boolean }) {
    // Initializes the core loop without UI concerns
    this.agent = await Agent.create(config);
  }

  /**
   * Execute a financial research query asynchronously.
   * Yields text streams, tool executions, and reasoning blocks.
   */
  async *executeResearch(query: string) {
    const stream = this.agent.stream(query);

    for await (const event of stream) {
      if (event.type === 'responding') {
        yield { type: 'text', content: event.text };
      } else if (event.type === 'tool_start') {
        yield { type: 'tool_call', name: event.tool, args: event.args };
      } else if (event.type === 'done') {
        yield { type: 'final_answer', content: event.answer };
      }
    }
  }
}
```

## Minimal Integration Surface

1. **Instantiate `Agent` Headless**: Bypass `src/index.tsx` and `src/cli.ts` entirely.
2. **Hook into the Event Stream**: The `Agent` class already yields structured events (`tool_start`, `thinking`, `done`). A Lumina adapter simply maps these events to Lumina's internal routing format.
3. **Inject Lumina's Tools**: Instead of only using Dexter's built-in registry (`src/tools/registry.ts`), the adapter should allow passing Lumina's native tools into the `Agent.create(..., tools)` initialization so Dexter can execute Lumina-native actions.

## Risks and Maintenance Concerns

*   **Version Drift in LangChain**: Dexter heavily relies on `@langchain/core` which has a high API churn rate. If Lumina uses a different version of LangChain internally, there will be peer dependency conflicts. **Mitigation**: Isolate the adapter in a separate service or Docker container, exposing it via MCP or REST, rather than directly importing it into the Lumina monolith.
*   **Dependency Bloat**: Dexter brings in `playwright`, `better-sqlite3`, and `whatsapp-baileys`. **Mitigation**: If importing directly, use a bundler configuration to strip out `src/gateway` and `src/tools/browser` if they are not needed.
*   **Prompt Coupling**: Dexter's logic is tightly coupled to `SOUL.md`. Overriding this behavior might degrade the agent's effectiveness. It is recommended to use Dexter "as-is" for financial research, rather than trying to make it a general-purpose agent.
