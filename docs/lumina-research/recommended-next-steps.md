# Recommended Next Steps

Based on the architectural analysis, dependency risks, and alignment with Lumina OS V3, here is the recommended path forward for integrating Dexter's capabilities.

## Strategic Decision: Partially Extract & Wrap

Lumina should **not** use the repository directly as a whole, nor should it completely rewrite it. The ideal approach is to **Extract** the core data-gathering tools and **Wrap** the cognitive reasoning engine via an Adapter.

### 1. Extract the Memory Layer (Priority: High)
*   **Action**: Rebuild or directly port `src/memory/search.ts` and `src/memory/indexer.ts` into Lumina's core "persistent project continuity layer".
*   **Why**: The implementation of Hybrid Search with Temporal Decay and MMR is highly sophisticated and exactly what Lumina needs for long-term project memory.
*   **Complexity**: Medium. Relies on SQLite, so it's easy to lift and shift into Lumina.

### 2. Wrap the Financial Agent (Priority: Medium)
*   **Action**: Create a service wrapper (e.g., via Model Context Protocol or a standard REST API) around `Agent.create()` and the `src/agent/` directory. Strip away the CLI (`src/cli.tsx`) and Gateway (`src/gateway/`) directories.
*   **Why**: Dexter's financial analysis prompts (`SOUL.md`) and specialized skills (`src/skills/x-research/`) are excellent. Wrapping it allows Lumina to delegate financial tasks to a "Financial Specialist Node" without contaminating Lumina's core with LangChain dependencies or Playwright binaries.
*   **Complexity**: Low. Requires building a simple Express/Hono server or MCP layer on top of `src/agent/agent.ts`.

### 3. Rebuild the Skills Framework (Priority: High)
*   **Action**: Study `src/skills/loader.ts` and `src/skills/registry.ts`. Rebuild a generalized version of this Markdown-based agent-instruction framework for Lumina's "cognitive orchestrator".
*   **Why**: The ability to define discrete research loops (like X-research or DCF valuation) via structured Markdown files is an incredibly powerful orchestration pattern.
*   **Complexity**: Medium. Requires designing a standard parser that converts Markdown instructions into prompt templates and tool definitions.

### 4. Avoid Entirely (Priority: N/A)
*   **Action**: Do not attempt to integrate the React/Ink CLI (`src/index.tsx`, `src/cli.ts`) or the WhatsApp gateway (`src/gateway/`).
*   **Why**: These are tightly coupled delivery mechanisms that conflict with Lumina's design philosophy.

## Summary Matrix

| Component | Recommendation | Strategic Value | Implementation Complexity |
| :--- | :--- | :--- | :--- |
| **Agent Loop (`src/agent/`)** | Wrap (Adapter/MCP) | High (For Finance) | Low |
| **Prompts (`SOUL.md`)** | Study & Adapt | High | Low |
| **Memory (`src/memory/`)** | Extract & Port | Critical | Medium |
| **Skills System (`src/skills/`)** | Rebuild Internally | Critical | Medium |
| **CLI & UI** | Avoid | None | N/A |
| **WhatsApp Gateway** | Avoid | None | N/A |
