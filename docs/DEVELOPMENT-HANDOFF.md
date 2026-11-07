# Agent Workspace Development Handoff

Reviewed 2026-11-07. Architecture specification; no runtime service is claimed to exist.

## Product definition and UX

Agent Workspace is a run-centric problem-solving environment, not a generic chat clone. Users set a goal, inspect/edit a plan, select provider/model, grant bounded tools, monitor execution and receive structured artifacts with evidence.

Surfaces: workspaces/projects; composer with provider/context/tools/budget; editable plan DAG; run timeline; artifacts and diffs; pause/resume/retry/fork checkpoints; MCP catalog; settings for provider profiles, key rotation, defaults, budgets, sessions and data deletion; operations console.

## Run state machine

created → planning → awaiting_approval → running → paused → succeeded/failed/cancelled/expired. Transitions are server-validated. Each run/step has stable idempotency identity. Browser disconnect must not terminate durable work. Cancellation propagates to streams/tools where possible.

## Runtime architecture

API/auth/quota; planner and typed DAG; scheduler with bounded parallelism/deadline; context compiler with ACL/provenance; provider-neutral gateway; deterministic tool broker; queue/workflow worker; checkpoint/event store; immutable artifact store; evaluator; redacted observability.

Do not execute arbitrary shell/code inside API worker. Separate sandbox service is a later independently threat-modelled and approved component.

## BYOK provider profiles

Users save provider credentials once and select a default per workspace/run.
- Envelope encryption; master key only in secret manager/KMS.
- Plaintext never returned after save; masked metadata only.
- Store provider, label, masked suffix, key version, owner, timestamps and status.
- Decrypt only inside provider adapter; never log, trace, export or put key into prompt/artifact.
- Verify with minimal request where supported; user controls model allowlist and spend cap.
- Rotation/revoke/delete invalidates caches and pending jobs.
- Provider inference charges are paid to provider; platform execution/storage/tool credits may still apply and must be disclosed.
- No silent fallback from BYOK to platform-funded key.
- Do not share secrets across workspace members without explicit audited organization feature.

## Model gateway and routing

Normalize request, stream chunks, usage, errors, cancellation and retry hints. Record provider/model/version, pricing version, tokens, latency and normalized error. Never expose raw provider payloads unnecessarily.

Initial routing should respect user's selected model. Optional fallback must be transparent. Later routing may consider task type, privacy, latency, context size, cost and preference. Enforce input/output tokens, run/user budget, deadline, concurrency and circuit breaker. Unknown price fails closed or requests confirmation. Distinguish Clemo platform credits from BYOK provider cost.

## Tools and MCP

Tool registry contains schema, owner, version, scopes, data class, risk class and approval policy. Model may propose a tool; deterministic broker authorizes. Validate input/output, cap bytes/runtime, treat descriptions/results as untrusted.

Approval required for external writes, publishing, financial/account actions, destructive operations, bulk export and external communication. MCP connections are explicit and revocable. Validate OAuth issuer, audience/resource, scopes and expiry; never forward unrelated bearer token. Remote MCP SSRF protection includes URL policy, DNS/IP validation, redirect revalidation and egress filtering. STDIO servers run sandboxed. Persist redacted invocation metadata and output digest, not secrets or unnecessary payloads.

## Context and retrieval

Context records source ID, owner, ACL, timestamp, version, provenance and trust. Re-check ACL at retrieval and use time. Retrieved text is data, never system policy. Injection detection is only a signal, not a defense. Cite sources and indicate stale/inaccessible context. Cross-tenant retrieval tests block release.

## Clemo credit boundary

Agent Workspace does not own wallet/payment. It requests a Clemo-issued quote/reservation, executes within reserved budget and sends a signed idempotent completion event. Clemo validates reservation and settles its ledger. BYOK provider spend is reported separately. Fail closed when reservation cannot be validated for billable platform service.

## Data model and reliability

Entities: workspace, membership, conversation, message, run, run_step, run_event, checkpoint, artifact, provider_profile, encrypted_secret_reference, model_policy, tool_server, tool_grant, approval, usage_record and audit_event. All tenant-scoped.

Queue-backed execution; retry only safe transient operations; dead-letter and operator replay; durable checkpoints; cancellation; provider health; budget enforcement; redacted traces; configurable retention; user export/deletion. No training on user data without explicit opt-in.

## Evaluation

Track task completion, step correctness, tool precision/recall, citation support, policy violation, prompt-injection success, recovery, cost per successful task, p50/p95 latency, approval burden and abandonment. Compare pinned models/tools against simple baseline.

## Delivery phases

P0 scaffold, CI, typed schemas, auth and tenant model.
P1 streaming chat, conversation persistence, provider adapters and BYOK vault.
P2 run state machine, plan DAG, scheduler, checkpoints and timeline.
P3 tool broker, MCP lifecycle, permissions and approvals.
P4 artifacts, retrieval/context compiler, budgets and Clemo credit adapter.
P5 evaluations, observability, replay, admin and production hardening.

## Acceptance gates

Cross-tenant IDOR tests; secret encryption/redaction/rotation/revoke; provider timeout/stream disconnect/cancel/rate-limit; tool schema and approval bypass; prompt injection; MCP audience/state replay/SSRF/malicious output; durable resume/replay; duplicate events; budget reconciliation; export/deletion/retention. No plaintext key in logs, traces, analytics, errors or database.
