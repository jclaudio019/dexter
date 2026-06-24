# System Integration Blueprint

This document details the architectural blueprint for integrating advanced autonomous capabilities, reasoning-first memory, and a human-on-the-loop framework into the current repository (`dexter-ts`).

## 1. Current Architecture Trace
The current `dexter-ts` codebase is a CLI-driven AI agent built with TypeScript, React/Ink, and LangChain.

*   **Entry Points:** The system receives user inputs primarily through a CLI interface (`src/cli.tsx`, `src/index.tsx`) and an optional WhatsApp Gateway (`src/gateway/index.ts`).
*   **Execution Pipeline:** The agent orchestrates execution through a loop (`src/agent/agent.ts`) combined with an `AgentToolExecutor` (`src/agent/tool-executor.ts`). The agent relies on a `Scratchpad` (`src/agent/scratchpad.ts`) to maintain state within a single query.
*   **Modularity:** Tool integration is highly modular via the `src/tools/registry.ts`, which dynamically registers capabilities ranging from web browsing (`src/tools/browser/`) to environment filesystem access (`src/tools/filesystem/`) and financial data retrieval (`src/tools/finance/`).
*   **Memory Models:** Long-term conversation state is stored on disk via `LongTermChatHistory` (`src/utils/long-term-chat-history.ts`), and advanced retrieval operations use a hybrid text/vector system powered by SQLite (`src/memory/`).

## 2. Modular Integration Design
To support dynamic planning, self-validation, and deeper post-turn reflection, the following directory layout is proposed to keep components decoupled from the core agent loops:

```
src/
├── agent/                 # Existing core logic (agent.ts, scratchpad.ts)
│   ├── planning/          # NEW: Modules for upfront plan mapping and human-on-the-loop signatures
│   └── validation/        # NEW: Outcome parsers to dynamically validate step N before step N+1
├── memory/                # Existing hybrid memory system
│   └── dialectic/         # NEW: Post-turn reflection engines, user preference extraction, summary layers
├── skills/                # Existing declarative multi-step workflows (e.g. SKILL.md)
│   └── AbstractSkill/     # NEW: Standardized interfaces for decoupling tool traces and environment skills
├── tools/                 # Existing functional capabilities (finance, system, browser, search)
└── orchestrator/          # NEW: A master state machine managing the Clarification -> Approval -> Execution lifecycle
```

## 3. The Core Loop Specification

The execution model will transition from a single iterative loop to a structured state machine with the following flow:

**User Command -> Clarification Loop -> Plan Signature -> Hybrid Executor -> Step Execution & Self-Validation -> Failure Reflection -> Post-Turn Dialectic Memory Commits**

1.  **User Command**: A query is intercepted via `src/cli.tsx` or `src/gateway/`.
2.  **Clarification Loop**: The orchestrator evaluates the prompt's boundaries. If the command lacks specifics, it delegates to `src/tools/ask-user-question/` to prompt the user (What, Why, How).
3.  **Plan Signature**: The system outputs a macroscopic structured plan of execution. The loop pauses (`workingState = "approval"`) and requires a human approval signature before continuing.
4.  **Hybrid Executor**: Once approved, the executor initiates Step N from the DAG (Directed Acyclic Graph) of the macro-plan.
5.  **Step Execution & Self-Validation**: The `AgentToolExecutor` performs the tool invocation. Before returning control to the agent loop, a validation module evaluates the output schema and data sufficiency. If data is lacking, the loop remains locked on Step N.
6.  **Failure Reflection**: If Step N fails consistently, a fast reflection pass analyzes the error traces and alters the approach for Step N without tearing down the entire plan.
7.  **Post-Turn Dialectic Memory Commits**: Upon completion, the dialectic engine processes the scratchpad traces. It extracts persistent user preferences, fixes, and structural insights, and pushes them to the `src/memory/` SQLite database (e.g., updating the summary and dream layers).
