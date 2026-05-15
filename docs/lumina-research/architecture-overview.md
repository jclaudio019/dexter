# Architecture Overview

## High-Level Architecture
Dexter is structured as a CLI-based autonomous agent, primarily utilizing TypeScript, LangChain, and Bun. The core operation revolves around an iterative loop that interacts with various Language Models (LLMs) to perform specialized financial research.

- **CLI Interface**: Uses React and Ink (`@mariozechner/pi-tui`) to render a terminal user interface. The entry point is `src/index.tsx`, which delegates to `src/cli.ts` for handling interactive sessions.
- **Agent Loop (`src/agent/agent.ts`)**: An iterative tool-calling cycle with a maximum iteration limit (default: 10). The agent passes context using a full array of LangChain objects (`SystemMessage`, `HumanMessage`, `AIMessage`, and `ToolMessage`).
- **Context Management**: Context thresholds trigger "compaction". Anthropic-style message arrays leverage `cache_control` to reduce prompt caching costs. Older "tool results" within the context are cleared once token limits are approached.

## Execution Model
- Iterative execution is central. The LLM plans its required tooling, the system executes it (often concurrently for read-only tasks), captures results into a `scratchpad`, and injects this back into the LLM context.
- Streaming responses stream `AIMessageChunk` objects. For specific models like DeepSeek, it explicitly enables a "thinking mode".
- When context reaches maximum thresholds, Dexter implements an auto-compaction mechanism to summarize earlier findings.

## Memory Model
Dexter maintains persistency between agent cycles through a bespoke memory module (`src/memory/`):
- **Database (`src/memory/database.ts`)**: Uses SQLite (`better-sqlite3`) to store memories.
- **Embeddings & Search**: Uses embedding models (OpenAI or locally configurable via Ollama) to convert text into vector embeddings.
- **Indexer (`src/memory/indexer.ts`)**: Synchronizes and chunks long-term markdown session files.
- **Retrieval (`src/memory/search.ts`)**: Implements a hybrid search using both vector similarity and exact text matches (BM25 or similar weighting mechanism), coupled with Maximal Marginal Relevance (MMR) and Temporal Decay logic.

## Planning & Orchestration Structure
- **Prompts**: `src/agent/prompts.ts` constructs the system prompt dynamically, including rules and the "Soul" of the agent (`SOUL.md`), which enforces independence and rigorous financial disciplines.
- **Tools Registry (`src/tools/registry.ts`)**: Conditionally registers tools based on the active LLM. Financial queries flow through a `financial_search` tool that delegates down to APIs, while standard search uses `web_search` (Exa/Tavily) or a Playwright-based `browser` for scraping.
- **Skills System (`src/skills/`)**: Declarative logic flows stored in Markdown files (e.g., `x-research/SKILL.md`). The skills provide a structured workflow that the LLM invokes as a tool (e.g., `skill_name`).

## Dependency Graph
- **Runtime**: `Bun`
- **Orchestration**: `@langchain/core` / `@langchain/openai` / `@langchain/anthropic`
- **Persistence**: `better-sqlite3`
- **UI**: `@mariozechner/pi-tui` (Ink/React CLI)
- **Web Scraping**: `playwright`, `@mozilla/readability`, `linkedom`
- **Gateway**: WhatsApp integration via `@whiskeysockets/baileys`

## Key Subsystems
- **Agent Core**: `src/agent/` (execution loop, context limits, tool execution, micro-compaction)
- **Tools**: `src/tools/` (Financial Datasets APIs, Web Search, Playwright browser)
- **Memory**: `src/memory/` (Vector-based storage, SQLite tracking)
- **Model / Providers**: `src/model/llm.ts` and `src/providers.ts` abstract multi-model capability.
