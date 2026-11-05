# ADR 0001: Astro on Cloudflare Workers

Status: accepted (2026-11-05). Context: free-tier budget, edge latency, streaming. Decision: Astro + Workers (Pages is not supported by the Astro adapter). Consequences: Workers-only APIs; use Hyperdrive for Postgres; local dev runs on workerd.