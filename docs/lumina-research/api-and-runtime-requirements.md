# API and Runtime Requirements

## Required APIs
- **LLM Providers**: At least one primary LLM provider is required. Out of the box, `OPENAI_API_KEY` (for models like `gpt-4o`) or `ANTHROPIC_API_KEY` (for `claude-3.5-sonnet`) is required to drive the reasoning loop.
- **Financial Data**: To execute its primary research loops, `FINANCIAL_DATASETS_API_KEY` is required.

## Optional APIs
- **Web Search**: For general search capabilities, Dexter attempts to use `EXASEARCH_API_KEY`. It will fall back to `TAVILY_API_KEY` if Exa is not provided.
- **Alternative LLMs**: Google (`GOOGLE_API_KEY`), xAI (`XAI_API_KEY`), DeepSeek (`DEEPSEEK_API_KEY`), Moonshot (`MOONSHOT_API_KEY`), OpenRouter (`OPENROUTER_API_KEY`).

## Local-Only Capability Support
- **LLM**: Full support for local models via Ollama (`OLLAMA_BASE_URL`). This allows Lumina to run the reasoning engine entirely locally.
- **Memory/Embeddings**: The SQLite memory module can use local embedding models via Ollama to ensure complete privacy for memory vectors.

## Docker & Runtime Requirements
- **Runtime Engine**: Designed specifically for `Bun` (>= 1.0) due to its test runner, package manager, and execution speed.
- **Playwright**: If using the browser tools for scraping, a Chromium browser must be installed (`npx playwright install chromium`). In a Docker environment, a specific Playwright-ready base image or manual installation of browser dependencies (e.g., `libgbm1`, `libnss3`) is required.
- **SQLite**: `better-sqlite3` requires compilation of native addons. If run in a minimal Docker container (like Alpine), additional build tools (`python`, `make`, `g++`) must be present during the install phase.

## GPU / Model Requirements
- No explicit GPU requirements for the node application itself.
- If running Ollama locally on the same host, a GPU with sufficient VRAM (e.g., >= 16GB for an 8B class reasoning model) is highly recommended for acceptable execution speed.

## Authentication Requirements
- The core system currently utilizes a `.env` file or interactive CLI inputs for API keys.
- Gateway capabilities (WhatsApp) require device-pairing via QR code generation in the terminal.
