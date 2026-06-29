---
icon: lucide/wrench
---

# Register tools

Tools are catalog entries that the MCP runtime can expose and invoke. A tool can point to a REST endpoint, an MCP HTTP/SSE endpoint, a WebSocket endpoint, or a local stdio command when explicitly enabled.

The catalog is where a raw endpoint becomes an enterprise capability. A URL by itself is not enough. Arcessa also needs to know who owns it, who can see it, how credentials are injected, whether it can be cached, which policies apply, and what audit context to attach.

## What counts as a tool?

| Tool shape | Example | Why register it |
|---|---|---|
| REST endpoint | `https://api.company.com/customer` | Give AI clients controlled access to an existing internal API. |
| MCP upstream | `https://mcp.vendor.com/mcp` | Federate or proxy another MCP server behind Arcessa policy. |
| WebSocket upstream | `wss://tools.company.com/mcp` | Support MCP servers that speak over WebSocket. |
| stdio server | `stdio:/usr/local/bin/mcp-server` | Expose local/on-prem command-based MCP servers through the runtime. |
| Reverse-proxy tunnel | On-prem stdio server connected to gateway | Let a private network host publish a tool without opening inbound access. |

!!! tip "Register capabilities, not random URLs"

    A useful catalog entry should have a human-readable name, description, owner/team, visibility, auth posture, and at least one test call. If nobody can explain what it does, AI clients should not discover it.

## Minimal REST tool

```bash
curl -s http://localhost:8080/api/v1/tools \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "company_lookup",
    "description": "Look up customer metadata",
    "integration_type": "REST",
    "url": "https://api.example.com/company",
    "visibility": "team",
    "team_id": "<team-id>"
  }' | jq
```

After creating it, immediately verify discovery:

```bash
curl -s http://localhost:8080/mcp \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}' | jq
```

## MCP upstream tool

Use `integration_type=MCP` and the upstream MCP endpoint:

```bash
curl -s http://localhost:8080/api/v1/tools \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "github_search",
    "description": "Search GitHub through an upstream MCP server",
    "integration_type": "MCP",
    "url": "https://api.githubcopilot.com/mcp",
    "visibility": "public"
  }' | jq
```

## Tool design checklist

| Field | Guidance |
|---|---|
| `name` | Stable, lowercase, and meaningful. Avoid names like `test` or `api`. |
| `description` | Write for a developer deciding whether to call it. Include what it does and what it must not be used for. |
| `integration_type` | Use `REST` for plain HTTP APIs and `MCP` for protocol-aware upstreams. |
| `url` | Must pass SSRF and egress controls. Prefer HTTPS in shared environments. |
| `visibility` | Start narrower than public. Promote only after ownership and governance are clear. |
| `team_id` | Use for team-scoped tools and workspace filtering. |
| `cache_ttl` | Use only when the output can safely be reused. |
| Tags | Add ownership, environment, data classification, and lifecycle labels. |

## Upstream credentials

Credentials are encrypted at rest when `MASTER_KEY` is set.

```bash
curl -s http://localhost:8080/api/v1/tools/<tool-id>/auth \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{
    "auth_type": "bearer",
    "auth_value": "upstream-secret-token"
  }' | jq
```

!!! warning "Never return upstream secrets"

    List, get, export, UI, and support-bundle responses should redact or omit secrets. Treat any plaintext upstream credential in an API response as a security bug.

## Auth types

| Auth type | Use when |
|---|---|
| None | The upstream is public or already protected by network controls. |
| Bearer | The upstream expects `Authorization: Bearer ...`. |
| Basic | The upstream expects username/password style credentials. |
| Header | The upstream expects a custom API key header. |
| OAuth client credentials | Arcessa should mint and cache upstream tokens. |
| mTLS | Use for high-trust internal upstreams when configured. |

## Cache behavior

Response caching is opt-in per tool. It is useful for expensive but stable calls such as catalog reads, lookup tables, or deterministic vendor requests.

Do not cache:

- user-specific secrets
- rapidly changing operational state
- approval-sensitive actions
- destructive operations
- responses that depend on invisible caller context

When a cache hit happens, audit should still record the invocation and mark `cached=true`.

## Visibility checklist

- Use `public` for same-org broad visibility.
- Use `team` for workspace-local tools.
- Use `private` for owner-only experiments.
- Use unique names in tests.
- Use group-scoped tokens when a client should only see a subset of tools.

## Governance checklist

For production tools, add:

- At least one ownership tag or team.
- A policy decision for risky tools.
- A guardrail rule for secret or PII sensitive tools.
- A budget scope if calls can incur model or vendor cost.
- A test invocation through `/mcp`, not only direct upstream curl.

## Failure modes to test

| Failure | Expected result |
|---|---|
| Bad upstream URL | Rejected at create time or blocked by SSRF guard. |
| Timeout | Clean MCP tool error and audit event. |
| Wrong upstream credential | Upstream `401/403` surfaces as tool failure without leaking secret. |
| Bad response schema | Runtime reports a structured failure. |
| Tool hidden by tenant scope | User cannot discover or call it. |
| Policy block | No upstream call happens. |
| Approval required | Approval row is created and exact `tool_id` is preserved. |
| Guardrail block | Sensitive request/response is blocked according to rule. |

!!! success "Production-ready tool"

    A production-ready tool is discoverable by the right users, invisible to the wrong users, secret-safe, governed, tested through `/mcp`, and visible in audit after every call.
