# Agent Workspace — AI Build Context

**Last reviewed:** 2026-11-07  
**Truth status:** architecture/design phase. No agent runtime, provider adapter, BYOK vault, MCP broker, durable scheduler or sandbox is implemented.

## Mission
A run-centric problem-solving workspace where a user provides a goal, sees an editable plan, chooses model/provider and bounded tools, approves consequential actions, follows durable progress and receives evidence-backed artifacts. Not a generic chat clone and not a free-running autonomous agent.

## Product ownership
Agent Workspace owns conversations, run/step state, planner, scheduler, context compiler, provider profiles, encrypted BYOK references, tool/MCP grants, approvals, artifacts, traces and user data lifecycle. Clemo owns wallet/payment/credits. Agent Labs owns benchmark datasets/results. Portfolio publishes only curated public evidence. No shared database or cross-product secrets.

## UX contract
Simple default path; advanced plan editing and traces progressively disclosed. Show provider/model, budget estimate, data scope and tools before starting. Stream early. Explain waiting, retries, partial results and failure. Support pause/resume/cancel/fork. Show sources, assumptions, uncertainty and artifacts. Ask confirmation before writes, external communication, purchases, destructive actions or broad export.

## Runtime architecture
Initial API/control plane uses TypeScript and Hono on Workers. Provider adapters normalize streaming, usage, cancellation, retry and errors. Durable run state and events must be persisted before terminal claims. Queue/workflow execution is bounded and idempotent. Arbitrary code/shell execution must be a separate isolated sandbox service, not in API Workers.

## Core invariants
- Model proposes; deterministic policy authorizes; executor performs typed schema-validated action.
- User-selected model is respected; fallback is visible and opt-in.
- BYOK secret encrypted with envelope encryption; plaintext only inside provider adapter; no read-back/log/trace/prompt/artifact. No silent platform-key fallback.
- Tenant ACL enforced before retrieval and again at action time.
- Retrieved content/tool descriptions/results are untrusted data, never instructions.
- Every tool grant is scoped, revocable, time-bounded and tenant-bound.
- Approval binds exact action hash, arguments, target and expiry; replay or mutation invalidates it.
- OAuth issuer/audience/resource/scope/state/expiry validated; bearer token never forwarded to unrelated origin.
- SSRF defense validates scheme, DNS/IP, redirects and egress; deny loopback/private/link-local/metadata.
- Each step has stable idempotency identity. Duplicate queue delivery cannot duplicate external writes.
- Cancellation and client disconnect cannot corrupt durable state.
- Clemo usage reservation is authoritative; signed idempotent settlement cannot exceed quote. BYOK inference spend is separate.
- User data is not used for training without explicit opt-in; deletion/export includes traces, artifacts and queued work.

## Performance model
Separate API acknowledgement, queue wait, context compilation, retrieval, provider TTFT, ITL/TPOT, tool/approval latency and total run time. Report p50/p95/p99 and cost per successful task. Optimize stream-first, connection reuse, bounded concurrency, ACL-filtered retrieval, prompt/context budgets, safe prefix caching and task-aware routing. No model router or fallback should silently override explicit user choice. A fast but low-quality or unsafe answer is not a performance improvement.

## Security test matrix
Cross-tenant IDOR; prompt injection from retrieved docs and tool metadata; BYOK secret leakage; OAuth audience/state replay; SSRF/DNS rebinding/redirects; approval replay and argument substitution; MCP schema poisoning; duplicate event/queue delivery; provider timeout/429/malformed stream; cancellation/disconnect/restart; budget exhaustion; oversized payload and output; export/deletion propagation.

## Initial delivery order
P0: scaffold, CI, typed schemas, auth/tenant model, run/event persistence.
P1: provider-neutral streaming + BYOK secret lifecycle.
P2: durable run state machine, scheduler, checkpoints, resume and cancel.
P3: tool broker, MCP lifecycle, approvals and SSRF controls.
P4: artifacts, retrieval/context compiler and Clemo settlement adapter.
P5: evals, tracing, cost/latency dashboards and production hardening.

## Technology policy
TypeScript-first for Workers-native control plane. Python for offline model evaluation/data workflows. Rust only for profiled CPU-heavy isolated components. Go only for an independently operated service justified by sustained concurrency/native libraries. No self-hosted inference until GPU economics, throughput, quality and operational ownership are demonstrated. Model/provider selection is task-class-specific and benchmarked on quality × latency × cost × safety, not tokens/sec alone.

## Cost, privacy and operations
Keep platform overhead within approved budget. BYOK doesn't make platform execution free; disclose storage, tool and orchestration charges. Per-user/provider caps, deadlines, rate limits, circuit breakers and global kill switches. Redacted observability; no raw prompt logging by default. User-controlled retention, export and deletion. No production credentials or paid services without approval.

## AI builder workflow
Read the runtime/security handoff and ADRs, pick one bounded vertical slice, write failing behavior tests first, implement minimum, run full suite, threat-test trust boundaries, measure before optimizing, update contracts/docs/ADR and open PR with exact evidence. Do not mark planned as implemented.
