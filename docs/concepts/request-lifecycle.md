---
icon: lucide/git-branch
description: How MCP and A2A calls move through Arcessa from client to audit.
---

# Request lifecycle

This page follows real calls through Arcessa. It explains where authentication, catalog lookup, governance, upstream invocation, and audit happen.

## MCP tools call

``` mermaid
sequenceDiagram
  participant Client as MCP client
  participant Gateway as gateway :8080
  participant Identity as identity :8081
  participant Runtime as mcp-runtime :8085
  participant Inventory as inventory :8082
  participant Governance as governance :8083
  participant Upstream as upstream tool
  participant Bus as NATS
  participant Obs as observability :8084

  Client->>Gateway: POST /mcp + Bearer token
  Gateway->>Gateway: CORS, rate limit, token revocation, scope check
  Gateway->>Identity: optional external bearer exchange
  Gateway->>Runtime: proxy request
  Runtime->>Runtime: verify principal and MCP protocol
  Runtime->>Inventory: resolve visible tool with on-behalf tenant context
  Inventory-->>Runtime: tool metadata and redacted config
  Runtime->>Governance: evaluate policy, budget, guardrails
  Governance-->>Runtime: allow / block / require_approval
  Runtime->>Upstream: invoke HTTP/SSE/WS/stdio/MCP upstream
  Upstream-->>Runtime: result or error
  Runtime->>Governance: post-response guardrails
  Runtime->>Bus: publish InvocationEvent
  Bus->>Obs: consume event into ClickHouse
  Runtime-->>Client: JSON-RPC result with _meta.governance
```

## Where failures are supposed to happen

| Stage | Example failure | Expected behavior |
|---|---|---|
| Edge auth | Missing token | `401` with protected-resource metadata where relevant. |
| Scope gate | Token lacks `mcp:invoke` | `403`. |
| Revocation | Revoked jti-bearing token | Rejected; fail-closed if Redis posture requires it. |
| Tenant lookup | Tool exists in another org | `404`-style non-disclosure where direct IDs are involved. |
| Governance | Block policy | Governed denial with audit. |
| Approval | Require approval | Pending approval id, no upstream call yet. |
| Guardrail | Secret in request | Redact/block/flag according to rule. |
| Upstream | Timeout or reset | Clean tool error and audit event. |
| Response guardrail | PII in output | Redact/block/flag before returning to client. |
| Observability | NATS unavailable | Invocation should degrade without crashing the call path. |

## A2A invocation

``` mermaid
sequenceDiagram
  participant Client as caller
  participant Gateway as gateway :8080
  participant A2A as a2a :8086
  participant Governance as governance :8083
  participant Agent as upstream agent
  participant Push as push receiver
  participant Obs as observability

  Client->>Gateway: POST /api/v1/agents/{id}/invoke
  Gateway->>A2A: proxy with verified principal
  A2A->>A2A: load agent, tenant visibility, dialect
  A2A->>Governance: pre-scan and policy evaluation
  Governance-->>A2A: decision
  A2A->>Agent: message/send or SendMessage
  Agent-->>A2A: task result or stream updates
  A2A->>Governance: post-scan output
  A2A->>Obs: emit invocation event
  A2A-->>Client: task state, artifacts, or SSE stream
  Agent-->>Push: optional task push callback
  Push->>A2A: token-gated push update
  A2A->>Obs: emit push/audit event
```

## Decision metadata

Arcessa responses should make governance visible. For MCP calls, clients can inspect `_meta.governance` to understand whether a result was allowed, blocked, cached, guarded, or parked for approval.

Example shape:

```json
{
  "_meta": {
    "governance": {
      "decision": "allow",
      "latency_ms": 12,
      "guardrail": "flag",
      "cached": false
    }
  }
}
```

## Why audit is part of the lifecycle

The audit event is not an afterthought. It is how Arcessa answers production questions later:

- Which user or token made the call?
- Which org/team/workspace was active?
- Which tool or agent was invoked?
- Which policy decision applied?
- Did a guardrail fire?
- Was the result cached?
- How many tokens or dollars were attributed?
- Did the upstream fail, timeout, or return an error?

!!! tip "Debugging order"

    When a call fails, inspect the edge status first, then token scopes, then tenant visibility, then governance decision, then upstream transport, then observability ingestion.
