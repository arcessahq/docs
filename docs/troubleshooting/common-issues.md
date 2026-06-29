---
icon: lucide/wrench
---

# Common issues

## Port already in use

Stale service binaries can keep ports open.

```bash
lsof -ti tcp:8080-8086 | xargs kill -9
pkill -f 'enterprise-gateway/bin/' || true
```

## Frontend fails to build

Use Node 24:

```bash
nvm use 24
make test-frontend
```

## E2E gets 429

The gateway limiter can throttle one admin user during large e2e runs. The harness raises `RATE_LIMIT_PER_MIN`, but Redis may retain old counters.

```bash
docker exec egw-redis-1 redis-cli FLUSHALL
```

Container names may differ by compose project. Use `docker ps` if needed.

## Tool create returns SSRF error

Production posture blocks private, loopback, link-local, and metadata targets.

For local dev:

```bash
ALLOW_PRIVATE_UPSTREAMS=true
```

For production, prefer an explicit host allowlist:

```bash
UPSTREAM_ALLOWED_HOSTS=api.company.internal
```

## MCP client sees no tools

Check:

- Token has `mcp:invoke` and `catalog:read`.
- Tool is visible to the token's org, team, owner, or group restriction.
- Workspace header is not narrowing the catalog unexpectedly.
- Tool was registered through gateway, not only inserted directly in the DB.

## OAuth client fails token exchange

Check:

- Redirect URI matches the registered client.
- PKCE verifier matches the challenge.
- Authorization code was not already consumed.
- `resource` maps to an accepted audience.
- Client auth method matches discovery metadata.

## SSO user logs in with wrong role

Role claims may live in the access token, ID token, userinfo response, or nested Keycloak-style `realm_access.roles`.

Check:

```bash
SSO_ROLE_CLAIMS=roles,groups,realm_access.roles
SSO_ROLE_MAP=platform-admin:admin,tool-editor:editor
SSO_DEFAULT_ROLE=viewer
```

Then run the real provider matrix:

```bash
make real-up
make matrix-up
make matrix-identity
```

## Observability page is empty

Check:

- NATS is reachable.
- ClickHouse is reachable.
- The call path emits an `InvocationEvent`.
- The event carries org/team context for tenant-scoped reads.
- The UI is querying the same org/workspace as the invocation.

## Real stack browser SSO cannot reach Keycloak

For browser auth-code SSO, the issuer host must resolve from the browser:

```text
127.0.0.1 keycloak
```

Add that to `/etc/hosts` when using the real stack locally.
