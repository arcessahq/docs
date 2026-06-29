---
icon: lucide/key-round
---

# Auth model

Arcessa supports three common enterprise access patterns:

| Pattern | Best for |
|---|---|
| Arcessa-issued session JWT | Admin console and first-party web sessions. |
| Arcessa-issued API token or PAT | Cursor, Claude Desktop, CI, CLIs, scripts, and tools that only support static Bearer headers. |
| OAuth authorization code with PKCE | Clients that can do the full OAuth dance and need standard discovery. |
| External IdP bearer-token exchange | Enterprise clients that already have Okta, Entra, Keycloak, Dex, or Hydra access tokens and cannot mint Arcessa PATs. |

## Roles, permissions, and scopes

Roles describe broad human authority:

| Role | Intended capability |
|---|---|
| viewer | Read-only control-plane access. |
| editor | Catalog and agent mutation. |
| admin | User, role, policy, token, and system administration. |

Permissions are fine-grained write rights such as `tools.write`, `agents.write`, `policies.write`, and `roles.manage`. API token scopes are gateway-level route gates such as `mcp:invoke`, `catalog:read`, `agents:read`, and `observe:read`.

!!! important "Scopes and permissions are different"

    A scope gets the token through the edge route. A permission decides whether the backend mutation is allowed. A token usually needs both for write actions.

## Token posture

| Token type | Revocable | Typical lifetime | Typical use |
|---|---:|---|---|
| Session JWT | No jti by default | `JWT_TTL` | Browser session. |
| PAT | Yes | Capped by `PAT_MAX_TTL_DAYS` | Developer local tools. |
| Admin API key | Yes | Capped by `API_KEY_MAX_TTL_DAYS` | Automation and service accounts. |
| OAuth access token | Yes | OAuth config | Standards-based MCP clients. |
| External IdP exchanged token | Yes | `EXTERNAL_IDP_EXCHANGE_TTL` | Enterprise bearer-token bridge. |

When `REVOCATION_FAIL_CLOSED=true`, jti-bearing tokens fail closed if Redis cannot confirm revocation state.
