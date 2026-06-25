# Extraction Map

This document outlines the exact modules, files, and patterns that are most valuable for extraction or study when integrating with Lumina OS V3.

## Core Orchestration Logic
*   **`src/agent/agent.ts`**: The primary agent loop.
    *   **Study point**: How it builds messages using a progressive context array (Anthropic-style with `ToolMessage` injection).
    *   **Study point**: How it streams responses and distinguishes between "thinking" output, "tool calls", and standard "responding".
*   **`src/agent/compact.ts` & `src/utils/tokens.ts`**: Handles context thresholding.
    *   **Study point**: `getAutoCompactThreshold` logic prevents token exhaustion. When a threshold is hit, it pauses the loop to run a secondary LLM summarization call over older context.

## Memory Structures
*   **`src/memory/` (Entire Directory)**: A high-value extraction target.
    *   **`src/memory/database.ts`**: The SQLite abstraction for vector storage.
    *   **`src/memory/search.ts`**: `hybridSearch()` is particularly interesting as it implements exact-text weighting combined with embedding similarity, temporal decay (`temporal-decay.ts`), and Maximal Marginal Relevance (`mmr.ts`) for diversity in search results.
    *   **`src/memory/indexer.ts`**: Synchronizes Markdown file changes in the background and chunks them appropriately for embedding.

## Planner and Heuristics Patterns
*   **`SOUL.md`**: The base prompt directive. Highly effective mechanism for enforcing tone, behavioral constraints, and analytical strictness.
*   **`src/skills/`**: The "Skills" architecture.
    *   **`src/skills/x-research/SKILL.md`**: Shows how to wrap specific search patterns (e.g., combining operators for bullish/bearish signals) into an LLM tool. Lumina should adopt this pattern of defining "micro-agents" via markdown.

## Reusable Abstractions (Low Dependency)
*   **`src/providers.ts`**: A centralized registry of LLM providers mapping models, context windows, and fast-model variants.
*   **`src/tools/search/`**: Robust fallbacks between different web search providers (Exa vs Tavily).
*   **`src/utils/message-queue.js`**: A clean implementation for draining multi-message buffers.
