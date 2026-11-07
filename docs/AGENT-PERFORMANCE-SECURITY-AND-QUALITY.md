# Agent Workspace performance, security and output quality

**Reviewed:** 2026-11-07  
**Status:** runtime architecture and acceptance contract. No running agent service or measured latency is claimed.

## Product outcome

Agent Workspace is a run-centric problem-solving environment. The user should state a goal, understand the plan, approve bounded capabilities, observe progress and receive a verifiable result. It is not a generic chat clone. Simplicity belongs in the interface; correctness, provenance, cost and policy belong in the runtime.

## Consumer market and trust

2026 research points to growing AI use but a trust and control gap:
- Ipsos India AI Monitor reports >80% AI awareness among surveyed Indian consumers: https://www.ipsos.com/en-in/ipsos-ai-monitor-2026-india-report
- NIQ's top-eight-metro shopping study shows high AI shopping usage/openness, but is not representative of all India: https://nielseniq.com/global/en/insights/report/2026/agentic-commerce-india-2026/
- Gartner's 2026 US consumer research finds stronger acceptance of discovery and comparison than autonomous purchase decisions; trust and accuracy remain friction: https://www.gartner.com/en/newsroom/press-releases/2026-05-27-gartner-survey-finds-consumers-want-ai-shopping-help-but-not-ai-purchase-decisions
- Menlo Ventures' 2026 US consumer AI survey estimates consumer spend grew faster than adoption; its own survey scope and estimates must not be generalized to every geography: https://menlovc.com/perspective/2026-the-state-of-consumer-ai/

Implications: optimize for useful outcomes, not token volume; show sources and uncertainty; expose provider/model and cost; keep humans in control of consequential actions; make cancel/retry/resume predictable; disclose AI use and data retention; do not hide external provider costs.

## Runtime language and topology

Keep orchestration/API in TypeScript on the selected Workers/Hono platform until benchmarks justify another service. TypeScript provides native alignment with streaming Web APIs, Workers bindings and versioned Zod/OpenAPI contracts.

- Rust is suitable for a measured CPU-bound, isolated module or sandbox runtime, not default orchestration. Account for Wasm interop, bundle size, startup and operational complexity.
- Go is a candidate for a separately deployed long-running scheduler/executor if sustained concurrency, native libraries, CPU workloads or process isolation justify it. Do not add it merely because benchmarks show raw throughput advantages unrelated to end-to-end inference.
- Inference provider/model, prompt length, queueing, network distance, tool latency and retrieval often dominate user-perceived latency. Optimize these before rewriting the control plane.
- Keep arbitrary shell/code execution outside the API Worker. A future sandbox is a separate service with a threat model, strict egress, ephemeral identity and explicit budget approval.

Cloudflare language support: https://developers.cloudflare.com/workers/languages/  
Cloudflare Rust Workers and Wasm bundle considerations: https://developers.cloudflare.com/workers/languages/rust/ and https://developers.cloudflare.com/workers/runtime-apis/webassembly/

## Latency and quality measurement

Record the full critical path, with request/run IDs and redacted metadata:

1. Client submit → API acknowledgement.
2. Auth/policy/quota and reservation.
3. Queue wait/scheduler wait.
4. Context compilation/retrieval.
5. Provider request → time to first token (TTFT).
6. Inter-token latency (ITL/TPOT), total generation.
7. Tool invocation and approval wait.
8. Validation/evaluation and artifact persistence.
9. First useful output and final completion.

Required metrics: p50/p95/p99 for API acknowledgement, queue time, TTFT, ITL/TPOT, total run, tool latency, retry/cancel/recovery; tokens, provider, model revision, context tokens, cost, task success, citation support, policy violation and user abandonment. Separate platform overhead from provider inference.

Proposed launch targets (not yet measured):
- API acknowledgement p95 ≤250ms for valid requests in-region.
- Run status/event propagation p95 ≤500ms after durable event commit.
- Scheduler overhead p95 ≤150ms excluding provider/tool I/O.
- First streamed token target is provider/model/task-class specific. Establish a representative baseline first; do not promise one universal TTFT across models.
- Every stream sends a useful status/heartbeat while waiting on a slow provider; UI never blocks on final response.
- Cancellation becomes visible immediately and propagates best-effort to provider/tool; billing settlement follows actual usage and policy.
- No result is marked succeeded until required artifacts and terminal event are durable.

For model serving, measure TTFT, queue time, decode time and token throughput separately; vLLM exposes per-request metrics for these distinct stages: https://github.com/vllm-project/vllm/blob/main/docs/features/per_request_metrics.md

## Performance architecture

- Stream model output as soon as provider protocol permits; do not buffer the entire completion.
- Use provider adapters with normalized streaming, cancellation, usage and retry semantics.
- Maintain connection reuse, bounded timeouts and explicit per-provider concurrency limits.
- Keep the first response path small: validate, reserve, enqueue/dispatch and stream status; do not run expensive retrieval, tool discovery or evaluation serially when it can be safely parallelized.
- Compile only relevant context; deduplicate chunks; apply ACL filters before retrieval; use bounded top-k and token budgets; rerank only when evaluation shows net quality gain.
- Use prompt/prefix caching only for identical safe prefixes; cache keys include model, prompt version, tenant and policy. Never share private content across tenants.
- Route by task class, quality floor, privacy, region, deadline and budget. The selected user model is respected unless an explicit, visible policy allows fallback.
- Avoid speculative multi-provider retries. Retry only idempotent transient failures; respect Retry-After and per-run deadlines.
- Use bounded parallelism for independent DAG steps; serialise conflicting writes and maintain deterministic event order.
- Store checkpoints and event sequence durably; reconnecting clients resume from cursor/sequence rather than restart the run.
- Use batch inference for offline evaluations and embeddings, not interactive turns.
- Measure output quality per task; lower latency with hallucinations or unsafe tool calls is not an improvement.

## Agent and MCP security

- Model proposes; deterministic policy engine authorizes; executor runs only typed, schema-validated, scoped actions.
- Default deny for arbitrary shell, arbitrary browser navigation, unknown MCP servers, financial actions, deletes, deploys, permission changes and bulk exports.
- Treat retrieved documents, webpages, issues, code, tool descriptions and tool outputs as untrusted data. Prompt-injection detection is advisory, never the authorization boundary.
- Tool grants are explicit, narrow, revocable, tenant-scoped, time-bounded and bound to resource IDs. Approval binds the exact action hash, arguments, target and expiry.
- OAuth tokens require issuer, audience/resource, scope, expiry and state validation. Never forward a bearer token to an unrelated origin.
- SSRF: HTTPS allowlist, DNS/IP validation, deny loopback/private/link-local/multicast/metadata, revalidate redirects, egress filter and cap response bytes/content type.
- MCP tool/schema changes trigger review; record server identity, version/hash, scopes and approval policy.
- BYOK secrets use envelope encryption; decrypt only in provider adapter; no plaintext read-back; rotate/revoke; redact from errors, logs, traces, metrics and artifacts. Never silently fall back to platform-funded keys.
- Every approval, policy decision, tool invocation, external side effect and settlement is auditable.
- User data is not used for model training without explicit opt-in. Export and deletion semantics include queues, caches, artifacts, traces and backup expiry.

OWASP's 2026 Agent/MCP guidance: https://owasp.github.io/DevSecOpsGuideline/2-Process/2-2-Develop/2-2-5-AI-Agent-and-MCP-Security/  
NSA MCP security considerations (2026-05-20): https://www.nsa.gov/Press-Room/Press-Releases-Statements/Press-Release-View/Article/4496698/nsa-releases-security-design-considerations-for-ai-driven-automation-leveraging/

## Output quality and consumer UX

- Default to concise, structured outcomes with summary, evidence, assumptions, limitations and next action.
- Cite sources for retrieved claims; show stale or inaccessible sources rather than silently omitting them.
- Distinguish generated suggestion from verified external action. Show a confirmation preview before consequential writes.
- Provide progress, pause, resume, cancel, retry and fork; explain whether an operation is safe to retry.
- Expose provider/model, approximate token/credit cost and budget before expensive work.
- If uncertain, ask one precise question or return a bounded partial result with uncertainty—not invented facts.
- Preserve user control over data retention, model selection, BYOK profiles, tool grants and external actions.
- Optimize the simple path: sensible defaults, no mandatory DAG editing, advanced controls progressively disclosed.

## Reliability and billing invariants

- Run state transitions are server-validated and persisted.
- Every run step has stable idempotency identity; retries cannot duplicate external writes.
- Queue delivery is at-least-once; consumers deduplicate and use an outbox/inbox pattern.
- Cancellation, provider timeout, client disconnect and worker restart must not corrupt state.
- Clemo owns wallet/credits. Workspace reserves via signed quote, reports signed idempotent completion; BYOK provider costs are separate.
- Unknown price, invalid reservation or exceeded budget fails closed before billable platform execution.
- Terminal run state and usage settlement reconcile; unresolved events go to a DLQ and operator alert.

## Evaluation and release gates

Evaluate against deterministic baselines and held-out tasks. Metrics include task success, tool precision, invalid arguments, policy violation, citation support, injection success, recovery, cost per successful task, TTFT/ITL and user abandonment. Stratify by task class, model, context size and tool type.

Required security tests: cross-tenant access; direct/indirect prompt injection; key exfiltration; malicious MCP metadata/output; SSRF and DNS rebinding; OAuth audience/state replay; approval replay/hash mismatch; duplicate side effects; queue redelivery; cancellation; budget exhaustion; oversized output; deletion propagation.

Required reliability tests: provider 429/5xx/timeouts; malformed stream; disconnect/reconnect; partial output; duplicate events; stale checkpoints; worker restart; retry storm; concurrent DAG branches; conflicting writes; model fallback disabled/enabled.

Do not claim "fastest" or "best" without published workload, baseline, sample size, hardware/provider, model revision, p50/p95/p99, quality and cost.

## Delivery order

P0: scaffold, CI, typed contracts, auth/tenant model, secret handling and event persistence.  
P1: streaming conversation and BYOK provider adapter with redaction tests.  
P2: durable run state machine, scheduler, checkpoint/resume and cancellation.  
P3: tool broker, MCP lifecycle, approval binding and SSRF controls.  
P4: artifacts, retrieval/context compiler, budget/Clemo settlement adapter.  
P5: eval harness, tracing, load tests, staged rollout and incident playbooks.
