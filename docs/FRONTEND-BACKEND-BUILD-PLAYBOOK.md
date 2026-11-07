# Agent Workspace Frontend and Runtime Build Playbook

**Status:** design/build contract; no runtime is claimed to exist yet.

## Product experience

Agent Workspace is a run-centric AI problem-solving environment, not a generic ChatGPT clone. Its differentiator is inspectability and control: plans, steps, model/provider selection, tool calls, approvals, artifacts, traces, cost and recovery are visible and understandable. The UI must never imply a tool ran, a task completed or a source was checked unless the runtime provides evidence.

Primary screens: conversation/workspace list; new task composer; run plan and live execution; step/tool detail; approval queue; model/provider settings; BYOK profile lifecycle; artifacts; trace/cost inspector; usage/privacy settings. Mobile must support reading, approvals and cancellation, even if complex trace inspection is optimized for desktop.

## Frontend architecture

- Next.js App Router, React, strict TypeScript and Tailwind CSS v4 on an independent Vercel deployment.
- Server Components for shell and non-sensitive static content; client components for streaming, composer, run state, approval controls, model picker and trace visualization.
- Motion for React for state transitions and subtle feedback. GSAP only for a rare explanatory timeline; Three.js is not part of the core run interface.
- Accessible primitives; shadcn/ui patterns/Radix selectively. Strong information hierarchy, clear state labels, compact data tables, readable code and predictable keyboard shortcuts.
- Zod schemas and generated API/event types from OpenAPI/JSON Schema.
- Vitest + React Testing Library, Playwright, axe-core, SSE contract tests and visual regression checks.

Use a calm, high-density professional workbench aesthetic. Avoid decorative 3D, glowing gradients, excessive card nesting, fake terminal output, ambiguous agent activity and charts without units/denominators. Make running, waiting for approval, blocked, retrying, failed, canceled and completed visually distinct and screen-reader announced.

## Taste Skill and Impeccable

Install into the frontend repository:

```bash
npx skills add https://github.com/Leonxlnx/taste-skill --skill "design-taste-frontend"
npx impeccable install
```

Reload Claude Code. Initialize Impeccable with `/impeccable init`; keep PRODUCT.md / DESIGN.md accurate. Enable hooks only after reviewing the hook behavior and approving permissions: `/impeccable hooks on`.

Use Taste Skill for art direction and visual structure. Use Impeccable for critique, audit, polish, responsive/accessibility review and deterministic anti-pattern checks. This playbook's product semantics and accessibility rules take precedence. Don't apply the same redesign prompt to both skills in parallel. Browser checks need Node 22+ and Chrome/Edge/Chromium. Sources: https://github.com/Leonxlnx/taste-skill, https://www.tasteskill.dev/docs/, https://impeccable.style/docs/.

## Figma: use it for the interaction model

Figma is recommended for the shell, composer, run timeline, approval dialog, provider settings and trace inspector. Prototype the run lifecycle (queued → planning → running → awaiting approval → resumed → completed/failed/canceled), responsive behavior, keyboard focus and empty/error states. Establish reusable tokens and components. Do not spend time polishing speculative features before the runtime contract stabilizes; keep Figma and code aligned by documenting component/state names.

## Runtime architecture

- Hono API on Cloudflare Workers; TypeScript-first control plane.
- SSE for streaming by default, with cancellation and backpressure; use WebSockets only if a measured bidirectional requirement warrants them.
- Durable run state machine and event log; queue-driven workers; bounded concurrency, deadlines, retry budgets and idempotency keys.
- Provider adapters isolate provider-specific request/response formats. Explicit model choice is honored; no silent fallback. Model capability registry and context compatibility are versioned.
- Tool broker enforces policy outside model output. MCP grants are scoped, revocable and tied to workspace/user/server/tool. Human approval binds to exact action and canonical argument hash; replay or argument substitution must fail.
- BYOK: session-only default; optional encrypted vault only after threat review. Decrypt only inside provider adapter; never read back, log, expose in browser, or insert into unrelated prompts/traces.
- Artifacts stored separately with ownership, MIME/size/hash and retention metadata. Arbitrary code execution never runs inside the API Worker; isolated ephemeral sandbox is a later separately threat-modeled component.

## Cross-product boundary

Agent Workspace owns conversations, run orchestration, provider profiles, BYOK secret references, MCP grants, artifacts and trace production. Clemo owns identity/workspaces and credits/commerce. Agent Labs owns evaluations and datasets. Portfolio publishes curated public evidence. No shared database, broad parent-domain cookie, raw prompt activity event or implicit context propagation. Usage settlement is signed, minimal and idempotent; Clemo independently validates its reservation.

## Security and privacy invariants

Treat prompts, retrieved documents, tool descriptions, MCP schemas, model outputs and web pages as untrusted. Deterministic server policy authorizes actions. Validate tenant membership, origin, OAuth audience/state, resource ownership, URL/DNS/IP/redirects, schemas, payload sizes, timeouts and output caps. Deny loopback/private/link-local/metadata network targets and redirects by default for custom provider endpoints. Rate-limit and budget every run. Redact credentials and private payloads from traces. User-controlled export/deletion and retention; no training without explicit opt-in.

## Delivery sequence

1. API/UI contracts, typed schemas, app shell, CI and health/ready checks.
2. Auth/session and tenant isolation.
3. Conversation persistence and provider-neutral streaming.
4. BYOK lifecycle and provider/model selection.
5. Durable run state, events, resume/cancel/checkpoints.
6. Tool broker, MCP lifecycle, approvals and SSRF protection.
7. Artifacts, retrieval/context compiler and Clemo usage settlement.
8. Evaluation, tracing, cost/latency visibility and hardening.

Build one vertical slice at a time. Tests must cover duplicate events, provider timeouts/429/malformed streams, cancellation/disconnect/restart, cross-tenant IDOR, BYOK leakage, prompt injection, approval replay and budget exhaustion.

## Verification

Browser: inspect real desktop/mobile render, console/network, SSE disconnect/reconnect, keyboard-only approvals, screen-reader state announcements, reduced motion and loading/error/empty states. API: contract, unit, integration and security tests. Measure TTFT, ITL/TPOT, queue wait, retrieval, tool/approval time, total duration and cost per successful task at p50/p95/p99. State workload, sample count and environment. Never infer model speed from tokens/sec alone.
