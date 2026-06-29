# Contributing to Arcessa Docs

This repository contains the public documentation site for Arcessa.

## Local setup

```bash
uv sync
uv run zensical serve
```

The local preview runs on Zensical's default address unless you pass `--dev-addr`.

```bash
uv run zensical serve --dev-addr 127.0.0.1:8008
```

## Build before opening a PR

```bash
uv run zensical build --clean
```

The build must complete with `No issues found`.

## Writing style

- Prefer runnable examples over conceptual prose.
- Put the outcome of a command before deeper explanation.
- Use tabs when the reader must choose between clients, providers, or environments.
- Use admonitions for warnings, security notes, and production-only behavior.
- Keep docs ASCII unless a source term or product name requires otherwise.
- Never include real credentials, API keys, tokens, customer data, or private hostnames.

## Structure

| Directory | Purpose |
|---|---|
| `docs/get-started/` | First-run and local development guides. |
| `docs/mcp/` | MCP client, catalog, and transport documentation. |
| `docs/a2a/` | Agent gateway flows. |
| `docs/identity/` | OAuth, tokens, SSO, SAML, SCIM, and IdP behavior. |
| `docs/governance/` | Policies, approvals, guardrails, and budgets. |
| `docs/observability/` | Audit, usage, SIEM, metrics, and sessions. |
| `docs/reference/` | API and configuration reference. |

## SEO and agent indexing

Keep these files in sync when the URL structure changes:

- `zensical.toml` `site_url`
- `docs/robots.txt`
- `docs/llms.txt`
- `overrides/main.html`

The canonical production URL is:

```text
https://arcessahq.github.io/docs/
```
