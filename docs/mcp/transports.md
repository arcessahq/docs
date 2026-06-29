---
icon: lucide/cable
---

# MCP transports

Arcessa accepts MCP clients through the gateway and dispatches upstream calls based on catalog configuration.

## Front-door transport

The public MCP endpoint is Streamable HTTP:

```text
POST /mcp
GET /mcp
DELETE /mcp
```

The runtime supports JSON-RPC requests, SSE event streams, progress notifications, resumability with `Last-Event-ID`, session ownership checks, protocol-version negotiation, and error-code conformance.

## Upstream dispatch

| URL shape | Upstream mode | Notes |
|---|---|---|
| `https://...` | HTTP or MCP Streamable HTTP | Default remote path. |
| `http://...` | HTTP or MCP Streamable HTTP | Dev or trusted internal only. |
| `wss://...` | WebSocket | Use for upstreams that expose MCP over WebSocket. |
| `ws://...` | WebSocket | Dev or trusted internal only. |
| `stdio:...` | Local subprocess | Disabled unless `ALLOW_STDIO_UPSTREAMS=true`. |

## Stdio safety

Stdio upstreams spawn local processes on the mcp-runtime host. Use them for local dev and tightly controlled internal deployments only.

```bash
ALLOW_STDIO_UPSTREAMS=true
STDIO_ALLOWED_COMMANDS=mcp-server,my-tool
```

## SSRF posture

Production should leave `ALLOW_PRIVATE_UPSTREAMS` unset. The SSRF guard blocks loopback, private, CGNAT, link-local, and cloud metadata access unless deliberately allowed. Link-local and metadata IPs remain blocked even when private upstreams are allowed.

```bash
ALLOW_PRIVATE_UPSTREAMS=false
UPSTREAM_ALLOWED_HOSTS=api.company.internal,10.10.0.42
```

## Resumability expectations

When a client drops an SSE connection, it can reconnect with `Last-Event-ID`. Arcessa should replay missed events for the same principal/session and refuse cross-principal replay.

```bash
curl -N http://localhost:8080/mcp \
  -H "Authorization: Bearer $TOKEN" \
  -H "Last-Event-ID: 12"
```
