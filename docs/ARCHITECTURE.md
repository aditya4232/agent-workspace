# Chat architecture (planned)
Astro/React UI -> chat Worker -> gateway `reserve/settle/refund` -> provider (server key or BYO) -> SSE stream -> trace row.
Provider layer: Vercel AI SDK core with failover. SSRF rules for custom base URLs: https only, port 443, no redirects, public DNS only, block own hosts and private/metadata ranges, timeouts, size caps.
Keys: session-only default; optional AES-GCM envelope-encrypted vault, never returned to the client.
Untrusted inputs: tool output, retrieved text, saved memory. Tool policy is deterministic (allowlist, argument validation, human approval for side effects).
Spikes before code: Agents SDK vs plain Workers + AI SDK; assistant-ui licence/fit; browser-direct CORS per provider.
