---
icon: lucide/play
---

# Quickstart

This quickstart runs Arcessa locally and connects a real MCP client or curl request through the gateway.

## Requirements

| Tool | Version |
|---|---|
| Go | 1.26 or newer |
| Node.js | 24 or newer |
| Docker | Current Docker Desktop or compatible engine |
| jq | Optional, used by curl examples |

## Start the stack

From the product repo:

```bash title="Backend and infrastructure"
cd /Users/pawan/enterprise-gateway
cp .env.example .env

make infra-up
make dev
```

In another terminal:

```bash title="Admin UI"
cd /Users/pawan/enterprise-gateway
nvm use 24
make web
```

Open `http://localhost:3000` and sign in with:

| Field | Value |
|---|---|
| Email | `admin@local.dev` |
| Password | `changeme123` |

!!! warning "Change bootstrap credentials outside local dev"

    `BOOTSTRAP_ADMIN_PASSWORD=changeme123` is only for local development. Production boot refuses the default password.

## Get a bearer token

```bash title="Login"
TOKEN=$(curl -s http://localhost:8080/api/v1/auth/login \
  -H 'Content-Type: application/json' \
  -d '{"email":"admin@local.dev","password":"changeme123"}' \
  | jq -r .token)
```

Confirm the gateway accepts the token:

```bash title="Current principal"
curl -s http://localhost:8080/api/v1/me \
  -H "Authorization: Bearer $TOKEN" | jq
```

## Register a simple REST-backed tool

For a quick local call, point a tool at a reachable HTTP endpoint. In dev, `.env.example` enables `ALLOW_PRIVATE_UPSTREAMS=true`, so loopback and private upstreams can be used.

```bash title="Create a catalog tool"
curl -s http://localhost:8080/api/v1/tools \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "httpbin_get",
    "description": "Demo HTTP tool through Arcessa",
    "integration_type": "REST",
    "url": "https://httpbin.org/get",
    "visibility": "public"
  }' | jq
```

## Call through MCP

Every call goes through the same route: gateway auth, rate limit, runtime dispatch, governance, guardrails, event emission, and audit.

=== "List tools"

    ```bash
    curl -s http://localhost:8080/mcp \
      -H "Authorization: Bearer $TOKEN" \
      -H 'Content-Type: application/json' \
      -d '{
        "jsonrpc": "2.0",
        "id": 1,
        "method": "tools/list",
        "params": {}
      }' | jq
    ```

=== "Call a tool"

    ```bash
    curl -s http://localhost:8080/mcp \
      -H "Authorization: Bearer $TOKEN" \
      -H 'Content-Type: application/json' \
      -d '{
        "jsonrpc": "2.0",
        "id": 2,
        "method": "tools/call",
        "params": {
          "name": "httpbin_get",
          "arguments": { "query": "hello" }
        }
      }' | jq
    ```

=== "Initialize"

    ```bash
    curl -s http://localhost:8080/mcp \
      -H "Authorization: Bearer $TOKEN" \
      -H "MCP-Protocol-Version: 2025-11-25" \
      -H 'Content-Type: application/json' \
      -d '{
        "jsonrpc": "2.0",
        "id": 3,
        "method": "initialize",
        "params": {
          "protocolVersion": "2025-11-25",
          "capabilities": {},
          "clientInfo": {"name": "curl", "version": "0.1"}
        }
      }' | jq
    ```

??? example "Raw tools/list payload"

    ```json
    {
      "jsonrpc": "2.0",
      "id": 1,
      "method": "tools/list",
      "params": {}
    }
    ```

## What to check in the UI

| Page | What to verify |
|---|---|
| Catalog | The new tool exists, is visible, and has no plaintext secret fields. |
| Console | The tool appears in Browse/Try and can be tested interactively. |
| Governance | Add a policy and confirm `_meta.governance` appears in responses. |
| Observability | The invocation appears in audit, usage, and session views. |

???+ success "Done means these four signals are visible"

    - The MCP response returned successfully or carried `_meta.governance`.
    - The Catalog page shows the tool.
    - The Console page can discover the tool.
    - Observability shows at least one invocation event.

## Stop the stack

```bash
make dev-down 2>/dev/null || true
make infra-down
```

If stale local binaries hold ports:

```bash
lsof -ti tcp:8080-8086 | xargs kill -9
```
