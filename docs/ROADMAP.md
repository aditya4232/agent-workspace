# Agent Workspace Roadmap

## Phase 0 — safe chat foundation
- Provider adapter interface and exact model ID configuration.
- Session-only BYOK, streaming, cancellation, limits and error states.
- Conversation ownership and deletion semantics.
- No tool execution enabled by default.

## Phase 1 — traceable model runs
- Versioned trace schema and span lifecycle.
- Token/latency/cost metadata with redaction.
- Sanitized export to Agent Labs only with explicit user consent.

## Phase 2 — deterministic tool policy
- Tool registry, JSON Schema validation and per-tool permissions.
- Approval workflow and idempotent execution.
- Fake-tool integration tests and prompt-injection evaluation.

## Phase 3 — MCP integrations
- User-trusted server registry, transport restrictions, scope display and revocation.
- SSRF-safe endpoint validation.
- Per-tool approval, timeout, output limits and audit trail.

## Phase 4 — workspace integration
- Tool Manifest registration for Clemo.
- Independent login first; OIDC/one-time launch only after security review.
- Minimal signed activity events; no raw chat transcripts.

## Phase 5 — optional key vault and isolated execution
- Threat model, envelope encryption, key rotation and access audit.
- Separate sandbox for arbitrary code/tool execution.
- Separate cost, security and privacy approval before release.

## Release gates
Cross-tenant isolation, prompt-injection, SSRF, key exfiltration, duplicate execution, approval replay, cancellation, budget exhaustion and deletion tests must pass. No public status upgrade without deployed evidence.
