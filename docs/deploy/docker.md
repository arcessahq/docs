---
icon: lucide/container
---

# Docker and real stack

The local dev stack is optimized for speed. The real stack is optimized for production-like validation.

Use local dev to build features. Use the real stack to prove production behavior.

## Local dev

```bash
make infra-up
make dev
make web
```

Local dev uses host binaries for services and compose-backed infrastructure.

Local dev is intentionally friendly:

- private upstreams are allowed
- stdio upstreams can be enabled
- bootstrap credentials are simple
- services run as host processes
- the UI runs in a developer mode

That is useful for development, but it is not a production readiness signal.

## Real-org production posture

```bash
make real-up
make real-verify
make real-obs
make real-security
make real-load
make real-chaos
make real-test
```

The real stack runs all services in containers with `ENV=production`.

Real stack changes the assumptions:

- `ENV=production` triggers secure boot checks.
- Services run as containers.
- SSRF is locked down.
- Keycloak provides real OIDC/SAML behavior.
- Prometheus and Grafana observe services.
- SIEM sink receives real events.
- Seed data creates multiple orgs, teams, roles, tokens, tools, policies, budgets, and agents.

## Real stack ports

| Component | Host port |
|---|---:|
| Gateway | 18080 |
| UI | 13000 |
| Keycloak | 18088 |
| Prometheus | 19090 |
| Grafana | 13001 |
| ClickHouse HTTP | 18124 |
| Postgres | 15442 |
| SIEM sink | 19099 |
| OTel collector | 14318 / 18889 |

## What the real stack includes

- All seven backend services.
- Next.js standalone frontend.
- Postgres, Redis, NATS, ClickHouse.
- Real MCP and REST upstreams.
- Keycloak for OIDC and SAML.
- Prometheus, Grafana, OTel collector.
- SIEM webhook receiver.
- Multi-tenant seed data.

## When to use each target

| Target | Use it for |
|---|---|
| `make real-verify` | Basic end-to-end health with real upstreams. |
| `make real-obs` | SSO, SIEM, Prometheus, and OTel validation. |
| `make real-security` | SSRF, IDOR, scope, secret-redaction, OAuth DCR checks. |
| `make real-load` | Sustained multi-tenant load with limiter enabled. |
| `make real-spike` | Sudden burst and recovery behavior. |
| `make real-ratelimit` | Abuser throttling without hurting control traffic. |
| `make real-volume` | Large catalog and event volume behavior. |
| `make real-soak` | Longer steady-state confidence. |
| `make real-chaos` | Dependency/service restarts and graceful degradation. |
| `make real-test` | Full real-stack gate. |

## Secrets

`deploy/.env.real` is generated from `env.real.example` and is gitignored. Add real third-party tokens there when running external upstream tests.

```bash
GITHUB_MCP_TOKEN=<pat>
make real-seed
```

## Teardown

```bash
make real-down
make real-down KEEP_DATA=1
```

## Production-readiness signal

The real stack is valuable because it catches bugs that unit tests usually miss:

- wrong public route mounts
- misconfigured CORS
- bad OAuth metadata
- IdP-specific role claim differences
- tenant leakage across direct-ID routes
- Redis/NATS outage behavior
- secret leakage in support bundles
- upstream timeout and retry behavior
- ClickHouse audit ingestion lag
