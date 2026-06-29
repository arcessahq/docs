---
icon: lucide/network
---

# Architecture

Arcessa is a control plane for AI tool access. Clients call one governed endpoint, and Arcessa decides what is discoverable, callable, allowed, transformed, cached, metered, and audited.

## Request path

``` mermaid
sequenceDiagram
  participant C as Client
  participant G as Gateway
  participant I as Identity
  participant R as MCP Runtime
  participant Inv as Inventory
  participant Gov as Governance
  participant Up as Upstream Tool
  participant Obs as Observability

  C->>G: POST /mcp with Bearer token
  G->>I: optional external-token exchange
  G->>R: proxied MCP request
  R->>Inv: resolve visible catalog object
  R->>Gov: evaluate policy, budgets, guardrails
  R->>Up: invoke upstream over HTTP/SSE/WS/stdio
  R->>Obs: publish invocation event through NATS
  R-->>C: JSON-RPC result with _meta.governance
```

## Ownership boundaries

| Module | Boundary |
|---|---|
| gateway | Edge concerns only: routing, auth at the public boundary, CORS, rate limiting, revocation, OAuth metadata, MCP proxying. |
| identity | Principals, memberships, tokens, SSO, SCIM, OAuth AS, external IdP exchange. |
| inventory | Catalog state and secret-at-rest handling for upstream credentials. |
| governance | Decisions: policies, approvals, guardrails, budgets, compliance, SIEM destination configuration. |
| mcp-runtime | MCP protocol behavior, upstream dispatch, per-call enforcement, event emission. |
| observability | Read models for audit, usage, sessions, SIEM forwarding. |
| a2a | Agent cards, tasks, protocol shaping, streaming, push, federation. |
| shared | Common auth, config, HTTP, metrics, Redis, Postgres, crypto, SSRF, events, guardrails, pricing. |

## Tenancy model

Arcessa scopes user-visible objects by organization, team, owner, and visibility:

| Visibility | Who can see it |
|---|---|
| `public` | Users in the same organization. |
| `team` | Users in the selected team. |
| `private` | Owner and admins. |

Service tokens can bypass tenant scoping for internal resolution, but user context is propagated with on-behalf headers when the runtime reads inventory for a human caller.

## Security boundaries

- The gateway is the only internet-facing backend service.
- Internal decrypted secret routes require service tokens, not admin user tokens.
- SSRF checks happen at create time and dial time for outbound control-plane URLs.
- API tokens carry scopes and permissions; session JWTs keep role-based behavior.
- Auditable actions publish a shared `InvocationEvent` so MCP and A2A show up in the same observability pipeline.
