---
icon: lucide/terminal
---

# Local development

Arcessa is a Go workspace plus a Next.js admin UI. The public surface is the gateway on `:8080`; backend services should stay internal.

## Services

| Service | Port | Responsibility |
|---|---:|---|
| gateway | 8080 | Public edge, auth, CORS, rate limit, token revocation, route table, MCP front door. |
| identity | 8081 | Login, JWT, RBAC, teams, tokens, OAuth AS, OIDC, SAML, SCIM. |
| inventory | 8082 | Catalog of gateways, tools, resources, prompts, roots, servers, tags. |
| governance | 8083 | Policies, approvals, guardrails, budgets, compliance, SIEM config. |
| observability | 8084 | Audit trail, usage rollups, sessions, ClickHouse reads. |
| mcp-runtime | 8085 | MCP JSON-RPC, upstream transports, governance application, event emission. |
| a2a | 8086 | Agent registry, cards, tasks, invocation routing, push, federation. |
| frontend | 3000 | Admin console and developer console. |

## Normal loop

```bash
cd /Users/pawan/enterprise-gateway
cp .env.example .env
make infra-up
make dev
```

Run the UI:

```bash
nvm use 24
make web
```

Run a specific backend service:

```bash
cd services/identity
go run ./cmd/identity
```

Run Go tests for one service:

```bash
cd services/identity
go test ./...
go test -tags integration ./...
```

## Adding a public route

The gateway is the only public surface. A new backend handler is unreachable until it is mounted in:

```text
services/gateway/cmd/gateway/main.go
```

Use this rule for `/api/v1/*`, `/scim`, `/.well-known/*`, `/mcp`, and A2A routes.

## Data stores

| Store | Used for |
|---|---|
| Postgres | Service-owned relational schemas. |
| Redis | Rate limiting, token revocation, OAuth codes, refresh rotation, budgets, response cache. |
| NATS | Invocation events and internal async pipelines. |
| ClickHouse | Audit, usage, and session replay data. |

NATS and Redis are optional in dev. Services degrade gracefully: event publishing no-ops when NATS is unavailable, and the limiter avoids hard failures during local work.

## Useful environment knobs

| Variable | Local default | Meaning |
|---|---|---|
| `ALLOW_PRIVATE_UPSTREAMS` | `true` | Allows loopback/private upstream URLs in dev. Leave unset in production. |
| `ALLOW_STDIO_UPSTREAMS` | `true` | Enables local stdio MCP upstreams. Disable in production unless tightly controlled. |
| `MCP_RUNTIME_REQUIRE_AUTH` | unset | Production defaults to auth-required for runtime `/mcp`. |
| `RATE_LIMIT_PER_MIN` | `600` | Gateway base rate limit. E2E raises this to avoid self-throttling. |
| `REVOCATION_FAIL_CLOSED` | `true` | Rejects jti-bearing tokens if Redis revocation checks fail. |

## Gotchas

- Use Node 24 for the frontend.
- Kill stale binaries before debugging port issues.
- Use unique test names for tools and agents.
- When e2e looks throttled, flush Redis or use the e2e rate-limit override.
- Do not put new service-only routes behind the public gateway mount.
