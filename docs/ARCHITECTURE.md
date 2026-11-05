# Architecture (planned)

## Request flow
1. Browser sends a message to the chat Worker (cookie session shared across `*.adityacuz.in`, verified with public keys).
2. Worker asks the gateway to `reserve(user, project, cost)`; on denial it returns a clear message and no model call is made.
3. Worker calls the provider (server key, or the user's own base URL + key) and streams tokens back over SSE.
4. Tool calls pass through the **tool policy** before execution; results are labelled untrusted.
5. On completion the Worker `settle`s (or `refund`s on error) and writes a trace row.

## Provider layer
Vercel AI SDK core with failover. Custom base URLs are validated against SSRF rules: https only, port 443, no redirects, public DNS only, block own hosts and private/metadata ranges, timeouts and size caps.

## Keys
- Default: session-only. Browser-direct where CORS allows, otherwise a stateless pass-through that never logs or stores the key.
- Opt-in vault: AES-GCM envelope encryption; keys are never returned to the client; deletable at any time.

## Memory and context
Rules and tool policy live outside compactable context and are re-checked after compaction. Saved memory is untrusted input. Observation masking keeps long tool outputs out of the prompt.

## Open spikes before code
Agents SDK vs plain Workers + AI SDK; assistant-ui licence and fit; browser-direct CORS per provider; Command Code model IDs via `/models`.