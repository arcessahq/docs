---
icon: lucide/lock-keyhole
---

# Production hardening

This checklist assumes the real production posture stack, not the permissive local dev defaults.

## Mandatory environment changes

| Area | Required action |
|---|---|
| `ENV` | Set `ENV=production`. |
| JWT | Use a strong `JWT_SECRET`; never use the dev value. |
| Secrets | Set `MASTER_KEY` from KMS, Vault, or a secret manager. |
| Admin | Change `BOOTSTRAP_ADMIN_PASSWORD`; production boot rejects the default. |
| CORS | Set explicit `CORS_ALLOWED_ORIGINS`; do not rely on wildcard with credentials. |
| SSRF | Leave `ALLOW_PRIVATE_UPSTREAMS` unset unless using a narrow allowlist. |
| Stdio | Keep `ALLOW_STDIO_UPSTREAMS=false` unless tightly controlled. |
| Revocation | Keep `REVOCATION_FAIL_CLOSED=true` for jti-bearing tokens. |

## Network posture

- Expose only the gateway and frontend.
- Keep identity, inventory, governance, observability, mcp-runtime, a2a, Postgres, Redis, NATS, and ClickHouse private.
- Configure `TRUSTED_PROXY_CIDRS` so X-Forwarded-For is accepted only from trusted load balancers.
- Enforce egress policy for upstream tool URLs.

## Container posture

For each app container:

- Run as non-root.
- Drop Linux capabilities.
- Use `no-new-privileges`.
- Prefer read-only root filesystem.
- Use `tmpfs` for writable temp paths.
- Set memory, pids, and CPU limits.
- Scan images before release.

## Auth and token posture

- Require audience-bound OAuth access tokens.
- Keep token lifetimes short.
- Use scoped PATs for local tools.
- Use admin API keys only for automation that needs admin surfaces.
- Reject empty self-serve token scopes.
- Test revoked token behavior with Redis up and Redis down.

## Secret leakage tests

Seed sentinel values such as:

```text
DO_NOT_LEAK_MASTER_KEY
DO_NOT_LEAK_UPSTREAM_TOKEN
DO_NOT_LEAK_SIEM_SECRET
```

Then assert they do not appear in:

- list/get API responses
- export responses
- support bundle zip
- rendered UI
- validation errors
- logs captured in support bundles

## Security scan commands

```bash
make real-up
make real-security
make real-scan
```

Expected tools include gitleaks for secrets, Trivy for image/SBOM scanning, and ZAP for HTTP security checks.
