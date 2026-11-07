# Agent Workspace Documentation

Status: architecture and research documentation. Runtime implementation is not yet present in this repository.

- [Architecture](ARCHITECTURE.md): system boundaries and components.
- [Security model](SECURITY-MODEL.md): tool policy and data isolation.
- [Roadmap](ROADMAP.md): delivery phases.
- [Agent runtime, harness, BYOK and MCP](AGENT-RUNTIME-HARNESS-AND-BYOK.md): run lifecycle, provider profiles, model routing, MCP authorization, sandbox boundary, observability and evaluation.

## Product boundary
Agent Workspace owns conversations, run orchestration, provider profiles, BYOK secret references, MCP grants and trace production. Clemo owns credits, payment and subscription ledger. Never put provider keys in Clemo, portfolio analytics or Agent Labs datasets. Cross-product usage is reported as signed, minimal usage events; Clemo validates and settles its own reservation.

## Verification
Do not describe planned capabilities as available. Before a feature is called operational, link its implementation, tests, deployment, health check and security review.
