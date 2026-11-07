# ADR 0002: Deterministic policy owns tool authorization

- Status: Accepted; implementation pending.
- Date: 2026-11-07.

## Context
An LLM can propose tool calls but cannot be trusted to authorize its own actions. Retrieved content and tool output can contain prompt injection.

## Decision
All tool calls pass through a server-side deterministic policy that checks identity, tenant, tool allowlist, JSON Schema, capability, resource scope, budget and approval requirements. High-impact writes require explicit human approval. The executor receives only the validated approved action.

## Consequences
- Model output is a proposal, not authorization.
- Tool output is untrusted and cannot modify policy.
- Tool execution must be isolated from the auth/billing control plane.
- Every decision is auditable with minimal redacted metadata.
