# Lumina Integration Analysis

This document analyzes the repository from the perspective of Lumina OS V3 to determine integration boundaries.

## Capabilities Lumina Should Use Directly
- **Agentic Workflows & Prompts**: Dexter's SOUL.md configuration and the structured execution prompts are sophisticated and clearly oriented around research methodology rather than generic LLM interactions. Lumina should adopt or directly wrap this system prompt generation pattern.
- **Skills System (`src/skills/`)**: The declarative Markdown-based skills system (e.g., `x-research`) provides a highly extensible way to chain complex search requirements. The ability to give an agent a heuristic-based workflow is exactly aligned with a cognitive orchestrator.
- **Tool Execution Structure**: The `AgentToolExecutor` and its ability to manage concurrent vs blocking execution of read-only tools is robust.
- **Memory Chunking & Indexing (`src/memory/`)**: The implementation of vector search with Temporal Decay and MMR logic provides an excellent reference or direct-use capability for a persistent project continuity layer.

## Capabilities that Should Remain External
- **Gateway**: The WhatsApp integration via `@whiskeysockets/baileys` (`src/gateway/`) is a channel-specific delivery mechanism. Lumina should remain channel-agnostic at its core.
- **CLI Interface**: The React-based CLI (`@mariozechner/pi-tui`) in `src/cli.ts` and `src/components/` is tightly coupled to terminal interactions. This should be ignored.

## Capabilities to Eventually Extract Internally
- **Financial Data Pipelines (`src/tools/finance/`)**: Dexter relies heavily on specific APIs (e.g., Financial Datasets). To become a true orchestration layer, Lumina should extract these tools into generalized "adapters" or "providers", making the system agnostic to the underlying financial data provider.
- **Search Abstractions (`src/tools/search/`)**: The fallback structure between Exa and Tavily is a good pattern to extract and standardize across all Lumina web-search operations.

## Tightly Coupled and Risky
- **UI and Agent Coupling**: There are aspects of event emission (`tool_start`, `thinking`) that are heavily tied to how the React CLI expects to stream updates. Isolating the agent engine from its UI updates will be necessary if Lumina intends to run these headless or via a different frontend.
- **LangChain Deprecation Risks**: The system heavily relies on `@langchain/core`. LangChain frequently deprecates internal APIs, making the core agent logic (`src/agent/agent.ts`) potentially brittle for long-term upgrades.

## Stable and Reusable
- **Multi-Model Provider Abstraction (`src/model/llm.ts`)**: The ability to seamlessly switch between OpenAI, Anthropic, DeepSeek, and Ollama with proper caching control.
- **SQLite Vector Memory**: The `better-sqlite3` hybrid search implementation is well-contained, low-dependency, and stable.
