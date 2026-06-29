---
icon: lucide/test-tube-2
---

# Testing overview

Arcessa uses a full pyramid plus real-provider matrices. The goal is not only line coverage; it is behavior coverage across protocol edges, auth branches, failure modes, and production posture.

## Core test targets

=== "Fast local"

    ```bash
    make test-unit
    nvm use 24 && make test-frontend
    ```

=== "Docker-backed"

    ```bash
    make test-integration
    nvm use 24 && make test-e2e
    ```

=== "Everything"

    ```bash
    make test-all
    ```

| Layer | Tooling | Purpose |
|---|---|---|
| Go unit | `go test` | Pure logic: auth, scopes, SSRF, config, guardrails, pricing, parsers. |
| Go integration | testcontainers-go | Stores and handlers against real Postgres, Redis, NATS, ClickHouse. |
| Frontend unit | Vitest + React Testing Library | UI transforms, hooks, API client behavior. |
| E2E | Playwright | Browser journeys and cross-service API flows against full stack. |
| Real stack | Docker Compose | Production posture with real containers and seeded org. |
| Matrix | Real providers and SDKs | MCP, A2A, OAuth/OIDC, SAML, SCIM, failure, contract interop. |

## Real-provider matrix

```bash title="Provider and protocol interop"
make real-up
make matrix-up
make matrix-all
```

| Suite | Proves |
|---|---|
| `matrix-identity` | SCIM RFC behavior plus OIDC interop with real IdPs. |
| `matrix-tokens` | Self-serve PAT scope, lifetime, ownership, and escalation controls. |
| `matrix-a2a` | A2A variants, push modes, UAID guard, FSM, approval, IDOR. |
| `matrix-a2a-sdk` | Python and JS SDK peer interop. |
| `matrix-a2a-tck` | Official A2A TCK MUST conformance against the peer. |
| `matrix-mcp` | Official TypeScript SDK server and strict MCP fixture interop. |
| `matrix-mcp-spec` | Front-door MCP hardening and OAuth protected-resource behavior. |
| `matrix-mcp-resume` | Streamable HTTP event replay and owner binding. |
| `matrix-failure` | Toxiproxy latency and reset handling. |
| `matrix-contract` | Public route table and response-shape contracts. |

## What to add when changing code

| Change | Required tests |
|---|---|
| New route | Route-table contract, auth/scope negative tests, handler success. |
| New store query | Migration integration, tenant-scope direct-ID tests, not-found tests. |
| New MCP method | JSON-RPC conformance, e2e over gateway, audit event assertion. |
| New A2A behavior | Task FSM, SDK peer shape, audit, guardrail, cancellation if async. |
| New SSO claim logic | Real IdP matrix with claim in access token, ID token, and userinfo variants. |
| New token behavior | Revocation, wrong audience, expired token, wrong scope, Redis outage. |
| New secret field | Secret sentinel e2e and support-bundle redaction test. |

## Failure modes to preserve

- Timeout
- TCP reset
- Bad schema
- Expired token
- Invalid signature
- Rate limiting
- Partial response
- Duplicate event
- Out-of-order event
- Retry exhaustion
- Cross-org IDOR
- Secret redaction
- Upstream SSRF attempt

???+ example "Minimum release-candidate gate"

    ```bash
    make test-all
    make real-up
    make real-test
    make matrix-all
    make real-scan
    ```
