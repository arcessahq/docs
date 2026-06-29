---
icon: lucide/sliders-horizontal
---

# Configuration

Configuration is environment-driven. Start from `.env.example` in the product repo.

## Core

| Variable | Meaning |
|---|---|
| `ENV` | `development` or `production`; production turns on secure boot checks. |
| `JWT_SECRET` | Shared HS256 secret for service JWTs and issued tokens. |
| `JWT_ISSUER` | Expected issuer. |
| `JWT_TTL` | Default token lifetime. |
| `MASTER_KEY` | Encrypts upstream secrets at rest. |
| `PUBLIC_BASE_URL` | Public base when used by OAuth/resource metadata. |

## Services

| Variable | Default |
|---|---|
| `GATEWAY_ADDR` | `:8080` |
| `IDENTITY_ADDR` | `:8081` |
| `INVENTORY_ADDR` | `:8082` |
| `GOVERNANCE_ADDR` | `:8083` |
| `OBSERVABILITY_ADDR` | `:8084` |
| `MCP_RUNTIME_ADDR` | `:8085` |
| `A2A_ADDR` | `:8086` |

## Stores

| Variable | Store |
|---|---|
| `DATABASE_URL` | Postgres |
| `REDIS_ADDR` | Redis |
| `NATS_URL` | NATS |
| `CLICKHOUSE_ADDR` | ClickHouse native |
| `CLICKHOUSE_HTTP_ADDR` | ClickHouse HTTP |

## Redis TLS

```bash
REDIS_ADDR=managed-redis.example.com:6380
REDIS_TLS=true
REDIS_TLS_CA_FILE=/etc/ssl/redis-ca.pem
REDIS_TLS_SERVER_NAME=managed-redis.example.com
REDIS_TLS_INSECURE_SKIP_VERIFY=false
```

## SSO and provisioning

| Variable | Meaning |
|---|---|
| `OIDC_ISSUER` | OIDC issuer URL. |
| `OIDC_CLIENT_ID` | OIDC client id. |
| `OIDC_CLIENT_SECRET` | OIDC client secret. |
| `SAML_IDP_METADATA_URL` | SAML IdP metadata URL. |
| `SAML_SP_CERT` | SAML SP cert PEM. |
| `SAML_SP_KEY` | SAML SP private key PEM. |
| `SCIM_BEARER_TOKEN` | Enables SCIM when set. |
| `SCIM_ORG_ID` | Org confined to SCIM provisioning. |

## External IdP bearer tokens

| Variable | Meaning |
|---|---|
| `EXTERNAL_BEARER_ENABLED` | Enables direct external bearer-token acceptance. |
| `EXTERNAL_IDP_ISSUER` | Trusted issuer. |
| `EXTERNAL_IDP_JWKS_URL` | Trusted JWKS URL. |
| `EXTERNAL_IDP_AUDIENCES` | Accepted audiences. |
| `EXTERNAL_IDP_SCOPES` | Scopes assigned to exchanged Arcessa token. |
| `EXTERNAL_IDP_EXCHANGE_TTL` | Lifetime of exchanged token. |

## Egress controls

| Variable | Meaning |
|---|---|
| `ALLOW_PRIVATE_UPSTREAMS` | Allows private/loopback upstreams in dev. |
| `UPSTREAM_ALLOWED_HOSTS` | Exact host or CIDR allowlist. |
| `EGRESS_ALLOWED_HOSTS` | Runtime egress allowlist. |
| `EGRESS_RATE_QPS` | Optional outbound rate limit. |
| `ALLOW_STDIO_UPSTREAMS` | Enables stdio upstreams. |
| `STDIO_ALLOWED_COMMANDS` | Restricts stdio command basenames. |
