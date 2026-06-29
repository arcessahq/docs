---
icon: lucide/braces
---

# API examples

These examples use the public gateway at `http://localhost:8080`.

## Login

```bash
TOKEN=$(curl -s http://localhost:8080/api/v1/auth/login \
  -H 'Content-Type: application/json' \
  -d '{"email":"admin@local.dev","password":"changeme123"}' \
  | jq -r .token)
```

## Current user

```bash
curl -s http://localhost:8080/api/v1/me \
  -H "Authorization: Bearer $TOKEN" | jq
```

## List tools

```bash
curl -s http://localhost:8080/api/v1/tools \
  -H "Authorization: Bearer $TOKEN" | jq
```

## MCP initialize

```bash
curl -s http://localhost:8080/mcp \
  -H "Authorization: Bearer $TOKEN" \
  -H "MCP-Protocol-Version: 2025-11-25" \
  -H 'Content-Type: application/json' \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "initialize",
    "params": {
      "protocolVersion": "2025-11-25",
      "capabilities": {},
      "clientInfo": {
        "name": "curl",
        "version": "0.1"
      }
    }
  }' | jq
```

## MCP tools list

```bash
curl -s http://localhost:8080/mcp \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/list","params":{}}' | jq
```

## Create API token

```bash
curl -s http://localhost:8080/api/v1/tokens \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "ci-mcp-read",
    "scopes": ["mcp:invoke", "catalog:read"],
    "expires_in_days": 30
  }' | jq
```

## OAuth metadata

```bash
curl -s http://localhost:8080/.well-known/oauth-authorization-server | jq
curl -s http://localhost:8080/.well-known/oauth-protected-resource | jq
```

## Health and metrics

```bash
curl -s http://localhost:8080/healthz
curl -s http://localhost:8080/metrics
```
