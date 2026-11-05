# chat

Agentic chat workspace behind **chat.adityacuz.in**. Status: planning, no code yet.

- Multi-provider: Command Code/DeepSeek by default, or bring your own OpenAI- or Anthropic-compatible base URL and key (session-only by default, never stored unless you opt in).
- Tool calls shown inline with timing; MCP client with per-request allowlists and approvals.
- Saved history you can delete (hard delete within 24h) and export; "don't save" mode.
- Every run is a trace you can open in Trace Lab (see `labs`).
- Credits and limits enforced server-side by the adityacuz platform; this repo holds the client and agent code only.

Docs: `docs/ARCHITECTURE.md`. Blueprint: https://github.com/aditya4232/Portfolio-Adityacuz.com (branch `blueprint/v2-docs`, `docs/blueprint`).
Licence: MIT.
