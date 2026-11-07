# Agent Workspace Security Model

## Threat model
Assets: provider keys, chat content, uploaded context, tool credentials, workspace membership, traces, quotas and external systems accessed by tools. Adversaries include malicious users, prompt-injected documents/pages, compromised MCP servers, malicious tool output, SSRF endpoints, cross-tenant attackers and abusive automation.

## Control plane and data plane
Control plane: auth, tenant membership, policy, quotas, provider config, tool registry and audit. Data plane: model execution, retrieval, tool calls and streaming. Isolate tool execution from the control-plane Worker in a sandbox with narrow capabilities. Model proposes; deterministic policy validates; human approval authorizes high-impact operations; executor performs only approved schema-bound action.

## Tool permission classes
| Class | Example | Default |
|---|---|---|
| Pure | calculator, formatting | Allow after schema validation |
| Read-only public | allowlisted public docs | Explicit allowlist |
| Private read | user's connected workspace | Scoped grant |
| External write | send message, create issue, modify repo | Human approval each operation |
| Destructive | delete, deploy, change permissions, financial action | Deny by default; separate confirmation |
| Arbitrary code | shell, unrestricted browser, arbitrary MCP | Disabled until isolated sandbox and threat review |

## Provider endpoint SSRF
For custom base URLs enforce HTTPS, allowed port, DNS resolution checks and IP-range denial for loopback/private/link-local/multicast/metadata. Revalidate after DNS changes; no redirects by default. Apply timeout, response byte cap and content-type allowlist. Never forward service credentials to user-supplied hosts. Test DNS rebinding and IPv4/IPv6.

## MCP controls
User explicitly installs/trusts a server. Record owner, transport, endpoint, auth mode, scopes, tool names and approval mode. Reject arbitrary endpoints that fail SSRF checks. Version schemas. Re-review server changes; revoke credentials on disconnect. Treat results as untrusted and sanitize rendered content.

## Data lifecycle
Raw chat/tool outputs excluded from logs by default. Traces use metadata and hashes; sensitive payload capture is opt-in and short-lived. User can export/delete conversations. Document immediate deletion vs cache/queue expiry vs backup/provider retention. A key vault requires envelope encryption, tenant isolation, rotation, audit and no key read-back endpoint.

## Required tests
Cross-tenant access, direct/indirect prompt injection, key exfiltration, malicious MCP output, SSRF/DNS rebinding, authorization bypass, replayed approval, duplicate execution, queue retries, stream cancellation, budget exhaustion, oversized tool output and deletion propagation.
