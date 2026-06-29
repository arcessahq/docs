---
title: OAuth and Tokens in Arcessa - PATs, API Keys, OAuth 2.1, and External IdP Bearer Exchange
description: Choose the right Arcessa authentication flow for MCP clients, local tools, CI, enterprise IdPs, OAuth-capable clients, and admin automation.
icon: lucide/badge-check
---

# OAuth and tokens

Arcessa can be the authorization server for MCP clients and can also mint scoped local tokens for clients that cannot do OAuth.

## Table Of Contents

1. [Which flow should I use?](#which-flow-should-i-use)
2. [Personal Access Tokens](#personal-access-tokens)
3. [OAuth Authorization Server](#oauth-authorization-server)
4. [The full OAuth dance](#the-full-oauth-dance)
5. [Resource-bound access tokens](#resource-bound-access-tokens)
6. [External IdP bearer-token exchange](#external-idp-bearer-token-exchange)
7. [2P, 3P, and local-tool patterns](#2p-3p-and-local-tool-patterns)
8. [Token review checklist](#token-review-checklist)

## Which flow should I use?

<div class="grid cards" markdown>

-   __Static Bearer only__

    ---

    Use a scoped PAT or admin API key.

-   __OAuth-capable client__

    ---

    Use authorization code with PKCE and protected-resource metadata.

-   __Enterprise IdP token__

    ---

    Use external bearer-token exchange against trusted JWKS.

-   __Admin console__

    ---

    Use first-party session login.

</div>

## Personal Access Tokens

PATs are for a human developer using local tools like Cursor, Claude Desktop, VS Code, curl, or CI experiments.

Recommended scopes:

| Use | Scopes |
|---|---|
| Invoke MCP only | `mcp:invoke`, `catalog:read` |
| Browse agents | `agents:read` |
| Read audit and usage | `observe:read` |
| Admin automation | Prefer admin API key with explicit scopes and short lifetime. |

PAT guardrails:

- Non-admins can mint only their own tokens.
- Empty scopes are rejected for self-serve tokens.
- Escalating to admin, `*`, or above-own role is rejected.
- Lifetime is capped by `PAT_MAX_TTL_DAYS`.
- Tokens are revocable and should have a `jti`.

## OAuth Authorization Server

=== "Authorization server"

    ```bash
    curl -s http://localhost:8080/.well-known/oauth-authorization-server | jq
    ```

=== "Protected resource"

    ```bash
    curl -s http://localhost:8080/.well-known/oauth-protected-resource | jq
    ```

```bash title="Dynamic client registration"
curl -s http://localhost:8080/api/v1/oauth/register \
  -H 'Content-Type: application/json' \
  -d '{
    "client_name": "local-mcp-client",
    "redirect_uris": ["http://127.0.0.1:3333/callback"],
    "grant_types": ["authorization_code", "refresh_token"],
    "response_types": ["code"]
  }' | jq
```

Authorization code flow:

1. Client creates a PKCE verifier and challenge.
2. Client sends user to `/api/v1/oauth/authorize`.
3. User signs in and consents.
4. Client receives `code`.
5. Client redeems `code` at `/api/v1/oauth/token`.
6. Client refreshes with rotating refresh tokens.

## The full OAuth dance

``` mermaid
sequenceDiagram
  participant Client as MCP client
  participant Browser as User browser
  participant Gateway as Arcessa gateway
  participant Identity as Identity/OAuth AS

  Client->>Gateway: GET protected resource metadata
  Client->>Identity: dynamic client registration
  Client->>Browser: open authorize URL with PKCE challenge
  Browser->>Identity: login and consent
  Identity-->>Browser: redirect with code
  Browser-->>Client: code
  Client->>Identity: token request with code verifier
  Identity-->>Client: access token + refresh token
  Client->>Gateway: POST /mcp with access token
```

The important production properties are:

- Authorization codes are one-time use.
- PKCE verifier must match the challenge.
- Refresh tokens rotate.
- Access tokens are audience-bound to the intended resource.
- Revocation state is checked for jti-bearing tokens.
- Discovery metadata accurately describes supported auth methods.

## Resource-bound access tokens

OAuth clients should pass a `resource` value for the protected resource they intend to call:

```text
resource=http://localhost:8080/mcp
```

Arcessa mints the value into the token audience and enforces accepted audiences at verification time.

## External IdP bearer-token exchange

Use this for enterprise systems that already get Okta, Entra, Keycloak, Dex, or Hydra access tokens and cannot use Arcessa OAuth.

```bash
EXTERNAL_BEARER_ENABLED=true
EXTERNAL_IDP_ISSUER=https://idp.example.com
EXTERNAL_IDP_JWKS_URL=https://idp.example.com/.well-known/jwks.json
EXTERNAL_IDP_AUDIENCES=api://arcessa
EXTERNAL_IDP_ROLE_CLAIMS=roles,groups,realm_access.roles
EXTERNAL_IDP_TEAM_CLAIMS=groups
EXTERNAL_IDP_DEFAULT_ROLE=viewer
EXTERNAL_IDP_EXCHANGE_TTL=15m
```

Required checks:

- Issuer must match.
- Audience must match.
- Signature must verify against JWKS.
- Expired, inactive, tampered, or wrong-audience tokens must fail.
- Role/team mapping must read access-token and ID-token claims, not only userinfo.

??? failure "Reject these tokens"

    - Wrong `iss`
    - Wrong or missing `aud`
    - Expired `exp`
    - Signature not found in JWKS
    - Tampered payload
    - Valid token for an inactive or unprovisioned principal

## 2P, 3P, and local-tool patterns

| Pattern | Meaning | Recommended Arcessa flow |
|---|---|---|
| First-party web UI | Arcessa's own admin console. | Session login. |
| Second-party internal automation | Your internal services, CI, or trusted platform jobs. | Scoped admin API key or service token. |
| Third-party MCP client | External or local client that supports OAuth. | OAuth authorization code with PKCE and DCR. |
| Local client without OAuth | Cursor/Claude/Desktop-style static config. | PAT with narrow scopes and bounded lifetime. |
| Enterprise bearer client | Client already has Okta/Entra/Keycloak token. | External IdP bearer-token exchange. |

## Token review checklist

| Question | Healthy answer |
|---|---|
| Does the token have a purpose-specific name? | Yes, e.g. `cursor-local-mcp` or `ci-catalog-read`. |
| Is the lifetime bounded? | Yes, especially for PATs. |
| Are scopes minimal? | Yes, no `admin` unless truly required. |
| Can it be revoked? | Yes, jti-bearing tokens use revocation checks. |
| Is audience enforced? | Yes for OAuth/resource-bound flows. |
| Is the owner clear? | Yes, human owner or automation owner is recorded. |
