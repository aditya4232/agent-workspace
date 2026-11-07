# Agent Workspace — inspectable agentic chat

Agent Workspace is the conversation runtime and agent execution product in the Aditya CUZ.in ecosystem. Users choose models/providers, inspect tool execution, control permissions and understand usage.

**Canonical app:** chat.adityacuz.in (planned)  
**Workspace launcher:** Clemo at clemo.adityacuz.in (planned)  
**Status:** design phase / planned. Features below describe intended behavior, not a live service.

## Product boundaries
- **Clemo:** identity/workspace shell, catalog, repository connections and launcher.
- **Agent Workspace:** chats, model/provider routing, streaming, tool policy, MCP, conversation state, user data retention and trace production.
- **Agent Labs:** independent experiment/evaluation system; consumes synthetic or approved traces.
- **Portfolio:** public narrative and published evidence; no access to private chats or keys.

## Capabilities
### Chat and model control
Provider-independent adapters for OpenAI-compatible and Anthropic-compatible APIs; model discovery where supported; explicit mid-thread model switch with context compatibility and cost warning; streaming/cancellation/retry; context budget and compaction transparency; session-only BYOK default. Encrypted vault is opt-in and only after threat review.

### Agent execution
Tool registry with workspace allowlist and JSON Schema validation; MCP client with server trust, transport restrictions and revocation; visible pending/approved/denied/running/succeeded/failed states; human approval before side effects, sensitive reads, external writes or costly actions; untrusted tool output; per-tool timeouts, output caps, concurrency limits and idempotency.

### Trace and observability
Trace model, retrieval, policy, tool and approval spans. Show provider/model, timing, tokens, estimated cost, retries, citations and policy decisions. Redact secrets/private payloads by default. Export sanitized traces to Agent Labs only with explicit user opt-in. Never send raw private chat content to Clemo's activity feed.

### Privacy
Session-only provider keys by default; opt-in encrypted vault only after security review; conversation export/deletion with clear backup/provider-retention semantics; no training on user data unless explicit opt-in and disclosure.

## Architecture
Hono/Cloudflare Worker owns API, authorization and policy boundary. UI is independently deployed Next.js. Streaming uses SSE by default unless WebSockets are justified; support cancellation/backpressure. Persist durable state explicitly; ephemeral runs expire. Provider calls are server-side when service-managed keys are used; keys are never logged or exposed to other users.

## Security
SSRF defense for custom provider endpoints: HTTPS, allowed ports, DNS/IP validation, block loopback/private/link-local/metadata ranges, no redirects by default, timeout and response size caps. Tool policy is deterministic and server-side; model output cannot authorize execution. Test prompt injection, cross-tenant isolation, key exfiltration, unsafe URLs, malicious MCP server and unbounded consumption.

See [Architecture](docs/ARCHITECTURE.md), [Security model](docs/SECURITY-MODEL.md), [Roadmap](docs/ROADMAP.md).
