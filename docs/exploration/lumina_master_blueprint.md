# Lumina OS Master Blueprint

This document represents the architectural blueprint for extracting components from the donor repositories into the `lumina-core` unified orchestrator.

## 1. The Upfront Conversation & Planning Interface (Lumina Gatekeeper)
* **Logic Location**: `src/cli.tsx` and `src/components/` in the CLI wrapper and `src/agent/agent.ts` the main agent loop.
* **Mechanism**: Use an Ink/React CLI wrapper or a WhatsApp gateway (`src/gateway/`) to gather initial parameters conversationally. The agent generates an initial plan through a dedicated prompt structure. The human user must review the output and explicitly approve it.

## 2. The Hybrid DAG Execution Engine (Source: dexter-free)
* **Logic Location**: `src/agent/agent.ts` (Agent loop), `src/agent/tool-executor.ts` (Tool invocation and concurrency).
* **Mechanism**: Dexter uses a loop architecture where LLMs emit step-by-step thinking/planning and tool execution via LangChain. A Hybrid DAG execution engine in Lumina would combine `Hermes-Agent`-style DAG structures with `dexter`'s Action/Validation loop. The agent maps the macro plan and evaluates the output of `tool_result` before continuing the loop. Concurrent execution of read-only tools is handled by `AgentToolExecutor`.

## 3. Dialectic & Reasoning Memory Infrastructure (Source: honcho & Hermes L4)
* **Logic Location**: `src/memory/` directory (Memory indexing, chunking, temporal decay, search logic).
* **Mechanism**: Dexter includes temporal memory retrieval and hybrid search. Honcho-like "Reasoning-first memory" logic can be synthesized by combining chunking/indexing of past session dialectics and extracting user preferences and goals via explicit summary/reflection layers. The memory system supports vector search combined with text scoring, augmented by `temporalDecay` features.

## 4. Financial Deep Research Skills (Source: dexter-free)
* **Logic Location**: `src/tools/finance/` directory (`api.ts`, `crypto.ts`, `earnings.ts`, `filings.ts`, `fundamentals.ts`, `get-financials.ts`, `get-market-data.ts`, etc.) and `src/skills/` (skills registry, e.g. DCF).
* **Mechanism**: Complex financial queries are decomposed and resolved using direct API calls for income statements, market cap, key ratios, earnings, insider trades. Tool results are appended to a scratchpad, allowing the agent to perform validation loops and read SEC filings (`read-filings.ts`).

## 5. Environment & System Tools (Source: OpenJarvis & OpenBrainAI)
* **Logic Location**: `src/tools/filesystem/` (`read-file.ts`, `write-file.ts`, `edit-file.ts`), `src/tools/browser/` (Playwright scraping), `src/tools/cron/` (scheduled execution), `src/tools/subagent/` (Spawn child tasks).
* **Mechanism**: Pure environment interaction skills exist via decoupled TypeScript tools. The file I/O tools read/edit files. The browser tool runs Playwright for web scraping. Shell execution and other primitives can be extracted cleanly without bringing along the `Ink` UI components.

## Unified Data Flow Map

**User Command -> Planning -> Honcho Memory Injection -> Hybrid Execution -> Tool Call -> Dexter Financial/System Verification -> Honcho Post-Turn Dialectic Dream/Reflection**

1. **User Command**: The user inputs a prompt via the Lumina Gatekeeper (CLI or Gateway).
2. **Planning**: The Gatekeeper parses intent and engages in an upfront conversation to finalize parameters. A macro plan is mapped out and requires human approval.
3. **Honcho Memory Injection**: Before execution begins, the system queries the Dialectic Memory Infrastructure. Context from prior summaries, user preferences, and "Dream layer" reflections is injected into the system prompt.
4. **Hybrid Execution**: The Hybrid DAG engine begins executing the first planned step. The DAG ensures dependencies are met.
5. **Tool Call**: The current execution node selects an environment tool (e.g., `src/tools/filesystem/read-file.ts` or a web search tool) to gather initial data.
6. **Dexter Financial/System Verification**: If the task involves financial research, tools from `src/tools/finance/` are triggered. The agent loop enforces an action/validation cycle, re-checking data against the original node's objective before unlocking the next DAG node.
7. **Honcho Post-Turn Dialectic Dream/Reflection**: Once the step (or session) concludes, the memory indexing system records the dialectic (the reasoning and tool results) and triggers asynchronous reflection to extract new preferences and update the summary/dream layers for future turns.

## Proposed Directory Structure
```
lumina-core/
├── gatekeeper/         # Upfront Conversation & Planning Interface (Lumina Gatekeeper)
├── execution/          # Hybrid DAG Execution Engine (Agent loops, Task dependency, concurrency)
├── memory/             # Dialectic & Reasoning Memory Infrastructure (chunking, vector db, temporal decay)
├── skills/             # High-level declarative skills (financial deep research logic)
├── tools/
│   ├── finance/        # Financial Data retrieval and verification
│   ├── system/         # Environment & System Tools (File I/O, Shell execution, Browser)
│   └── search/         # Web search and Perplexity
└── utils/              # Token counting, prompt building, caching, concurrency helpers
```
