---
title: Connect MCP Clients to Arcessa - Cursor, Claude Desktop, OAuth, and Bearer Tokens
description: Configure Cursor, Claude Desktop, curl, CI, and OAuth-capable MCP clients to connect through the Arcessa MCP gateway.
icon: lucide/plug-zap
---

# Connect MCP clients

Arcessa exposes one MCP endpoint:

```text
http://localhost:8080/mcp
```

Use a scoped token for clients that only support a static Bearer header. Use OAuth when the client supports authorization server discovery and browser consent.

## Table Of Contents

1. [Cursor with a Personal Access Token](#cursor-with-a-personal-access-token)
2. [OAuth-capable MCP clients](#oauth-capable-mcp-clients)
3. [Verify from curl](#verify-from-curl)
4. [Common failures](#common-failures)

## Cursor with a Personal Access Token

!!! tip "Best default for local tools"

    Use a PAT when the tool cannot open a browser for OAuth or cannot store OAuth refresh tokens safely. This is the right fit for most local Cursor and Claude Desktop setups.

Create a PAT in the admin UI:

1. Open `Access Tokens`.
2. Choose `New token`.
3. Select the scopes your client needs.
4. For a normal MCP client, start with `mcp:invoke` and `catalog:read`.
5. Copy the token once.

Configure Cursor:

=== "Cursor"

    ```json title="~/.cursor/mcp.json"
    {
      "mcpServers": {
        "arcessa": {
          "url": "http://localhost:8080/mcp",
          "headers": {
            "Authorization": "Bearer ${env:ARCESSA_TOKEN}"
          }
        }
      }
    }
    ```

=== "Claude Desktop"

    ```json title="claude_desktop_config.json"
    {
      "mcpServers": {
        "arcessa": {
          "url": "http://localhost:8080/mcp",
          "headers": {
            "Authorization": "Bearer ${env:ARCESSA_TOKEN}"
          }
        }
      }
    }
    ```

=== "Shell"

    ```bash
    export ARCESSA_TOKEN="<paste token>"
    ```

Use the same endpoint and Bearer header shape. Store the token in an environment variable or the operating system secret store when possible.

## OAuth-capable MCP clients

OAuth-capable clients should discover metadata from the gateway:

```bash
curl -s http://localhost:8080/.well-known/oauth-authorization-server | jq
curl -s http://localhost:8080/.well-known/oauth-protected-resource | jq
```

The OAuth flow supports dynamic client registration, authorization code with PKCE, refresh rotation, and resource-bound audiences.

## Verify from curl

```bash
curl -s http://localhost:8080/mcp \
  -H "Authorization: Bearer $ARCESSA_TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}' | jq
```

## Common failures

| Symptom | Likely cause | Fix |
|---|---|---|
| `401` | Missing, expired, revoked, or wrong-audience token. | Mint a new token; check OAuth `resource` or PAT scope. |
| `403` | Token passed edge auth but lacks scope or permission. | Add `mcp:invoke` or required route scope. |
| `429` | Gateway rate limit. | Use a higher tier or adjust `RATE_LIMIT_*` in dev. |
| Empty tools list | Tenant visibility, group restriction, or no catalog entries. | Check workspace, tool visibility, token groups, and catalog. |
