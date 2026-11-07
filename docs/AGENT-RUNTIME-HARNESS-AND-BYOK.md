# Agent Runtime, Harness, BYOK and MCP Architecture

Status: architecture proposal; no production runtime or provider integration is claimed.
Date: 2026-11-07.

## Product thesis

Agent Workspace is an inspectable problem-solving environment, not a chat transcript UI. The primary object is a run: goal, plan, bounded capabilities, tool calls, artifacts, checkpoints, approvals, cost and evidence-backed result.

## User experience

Workspace projects, threads, artifacts and saved workflows; composer with provider/model/context/tools/budget; editable plan with dependencies, cost/time/risk; run timeline with model steps, tool calls, retries, token/cost and trace IDs; artifacts with code diffs/files/data/citations; pause/resume/fork/compare checkpoints; settings for encrypted provider profiles, defaults, key rotation and deletion; MCP/tool catalog with per-tool permissions.

## Harness components

1. Run API validates identity, quotas, budget and idempotency.
2. Planner creates a typed task graph that users can inspect/edit.
3. Scheduler executes dependency-aware steps with bounded parallelism, deadlines and cancellation.
4. Model gateway provides provider-neutral request, streaming, usage and error contracts.
5. Context builder tags provenance, enforces ACLs, relevance, token budgets and redaction.
6. Tool broker validates schemas, policy, consent and output.
7. State store persists checkpoints, event log, artifacts and replay metadata.
8. Evaluator measures completion, citation correctness, tool correctness, policy violations, latency and cost.
9. Observability uses traces, metrics and redacted structured logs.
10. Artifact service stores immutable versions, checksums, retention and scoped download URLs.

Run state machine: created, planning, awaiting_approval, running, paused, succeeded, failed, cancelled, expired. Every transition is server-validated and recorded.

## Differentiation beyond chat

Goal-oriented multi-step tasks with editable plans; capability-based tool selection; provenance-aware context compilation; durable checkpoints; parallel calls with concurrency limits; structured artifacts/diffs; deterministic authorization before side effects; evaluator regression suite; replay from recorded non-secret inputs and fixtures; visible cost, latency and failure reason.

## BYOK provider profiles

Users save provider credentials once in profile settings and select a default per workspace.
- Provider adapters support compatible hosted and local/self-hosted APIs.
- Encrypt secrets with envelope encryption; master key exists only in secret manager/KMS.
- Store ciphertext, key version, provider, masked suffix, created/rotated/last-used timestamps and owner.
- Never return plaintext after save; show masked metadata.
- Decrypt only at provider request boundary; never log, trace, export or send key to model context.
- Verify server-side at save/test with a low-cost request where supported.
- Per-profile spend cap, model allowlist, rate limit and optional expiry.
- Rotation/revoke/delete is audited; deletion invalidates cache and pending jobs.
- BYOK means provider inference charges are billed directly by that provider; platform credits may still apply for platform compute, storage and tool execution. Explain this clearly.
- Never silently fall back from user's BYOK key to platform-funded key. Fallback is disabled unless explicitly enabled.
- Default design is server-side encrypted storage. Browser-direct key use, if ever offered, is separate and only for providers that explicitly support it.

## Routing and budgets

Router considers task class, context length, latency, cost ceiling, privacy policy, availability and user preference. Select only user-allowed providers/models and show model plus reason. Enforce max input/output tokens, per-run INR/USD budget, wall-clock deadline, retries and concurrency. Reconcile actual usage. Circuit-break provider failures; do not blindly retry non-idempotent tools. Version provider prices by effective date. Unknown price fails closed or requires confirmation. Distinguish platform credits from provider charges.

## MCP security

Follow current MCP authorization/security guidance; protocol support is not itself a security boundary.
- Each server connection records owner, origin, transport, identity, capabilities, scopes, data classification, approval mode and status.
- Explicit consent before connecting and before sensitive tool execution.
- OAuth 2.1/PKCE where applicable; validate issuer, audience/resource, expiry and scopes.
- Tokens are audience-bound to the MCP server; never pass unrelated upstream tokens through.
- Separate per-server credentials, short-lived where possible and encrypted at rest.
- Tool metadata/descriptions and outputs are untrusted, not policy instructions.
- Bind state handles server-side to authenticated principal; possession is not authentication.
- Deterministic per-tool policy and parameter validation outside the model.
- Human approval for external writes, financial actions, account changes, destructive operations, publishing and bulk export.
- SSRF protection: scheme/host allowlist, DNS/IP checks, block private/link-local/metadata ranges, revalidate redirects and egress at connection time.
- Sandboxed STDIO process with resource, filesystem and network restrictions and kill switch.
- Record tool name/version, redacted arguments, authorization decision, approval, output digest, timing and trace; never log credentials or sensitive payloads.

## Execution isolation

Never execute arbitrary code inside the web/API worker. If code execution becomes a feature, use a separate sandbox service with ephemeral VM/container isolation, non-root identity, seccomp/AppArmor or stronger isolation, resource/time limits, no host mounts, no production credentials, restricted egress, immutable patched images, per-run identity, artifact-only transfer and verified teardown. This requires separate threat model and budget approval.

## Reliability and data lifecycle

Event-sourced run timeline with idempotent step IDs; queue-based async execution; retry only safe transient errors; dead-letter queue and operator replay; cancellation propagation; user export/deletion across conversations/artifacts/provider metadata/indexes; configurable retention; no training without explicit opt-in; trace redaction; secret scanning; multi-tenant isolation tests.

## Evaluation metrics

Task success with independent graders, tool-call correctness, citation support, policy violation rate, prompt-injection success, recovery rate, p50/p95 latency, provider cost/run, platform cost/run, human intervention and abandonment. Never optimize success alone; pair with safety, cost and reproducibility.

## Sources

- MCP 2026-07-28 specification: https://github.com/modelcontextprotocol/modelcontextprotocol/tree/main/docs/specification/2026-07-28
- MCP security best practices: https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/docs/2026-07-28/tutorials/security/security_best_practices.mdx
- MCP Security Bench: https://arxiv.org/abs/2510.15994
- Security Considerations for AI Agents (2026): https://arxiv.org/abs/2603.12230
