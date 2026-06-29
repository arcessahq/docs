<p align="center">
  <strong>Arcessa Docs</strong>
</p>

<p align="center">
  Technical documentation for Arcessa, the governed control plane for MCP tools and A2A agents.
</p>

<p align="center">
  <a href="https://arcessahq.github.io/docs/"><strong>Docs</strong></a>
  ·
  <a href="https://arcessahq.github.io/web/"><strong>Landing site</strong></a>
  ·
  <a href="https://github.com/arcessahq/docs/actions/workflows/docs.yml"><strong>Deploy workflow</strong></a>
  ·
  <a href="CONTRIBUTING.md"><strong>Contributing</strong></a>
</p>

<p align="center">
  <a href="https://github.com/arcessahq/docs/actions/workflows/docs.yml">
    <img alt="Documentation" src="https://github.com/arcessahq/docs/actions/workflows/docs.yml/badge.svg">
  </a>
  <a href="LICENSE">
    <img alt="License: MIT" src="https://img.shields.io/badge/license-MIT-blue.svg">
  </a>
  <img alt="Built with Zensical" src="https://img.shields.io/badge/docs-Zensical-4f46e5.svg">
</p>

## What this is

This repository contains the public technical documentation for Arcessa.

The landing site lives at:

```text
https://arcessahq.github.io/web/
```

The docs site is intended to live at:

```text
https://arcessahq.github.io/docs/
```

Use the landing site for the product story. Use this repository for detailed technical guides, operator runbooks, integration walkthroughs, API examples, and production-readiness material.

## What the docs cover

| Area | Coverage |
|---|---|
| First-time onboarding | What Arcessa is, product tour, quickstart, and local development. |
| MCP | Connecting Cursor/Claude/CI, registering tools, upstream transports, Streamable HTTP behavior. |
| A2A | Agent registry, cards, tasks, streaming, push callbacks, federation, and gateway behavior. |
| Identity | PATs, API keys, OAuth 2.1, OIDC, SAML, SCIM, external IdP bearer-token exchange. |
| Governance | Policies, approvals, guardrails, budgets, trust and data-flow controls. |
| Observability | Audit, usage, session replay, metrics, OpenTelemetry, SIEM export. |
| Operations | Docker real stack, testing matrix, production hardening, troubleshooting. |
| Agents/search | `robots.txt`, `sitemap.xml`, `llms.txt`, Open Graph, Twitter card, canonical links. |

## Repository layout

```text
.
├── .github/
│   ├── ISSUE_TEMPLATE/
│   ├── workflows/docs.yml
│   └── pull_request_template.md
├── docs/                    # Markdown source pages
├── overrides/               # Zensical template overrides
├── zensical.toml            # Site metadata, navigation, theme, extensions
├── pyproject.toml           # Local docs toolchain
├── uv.lock                  # Locked Zensical dependency graph
├── CONTRIBUTING.md
├── SECURITY.md
├── SUPPORT.md
└── LICENSE
```

## Work locally

Requirements:

- Python 3.12+
- `uv`

Install and preview:

```bash
uv sync
uv run zensical serve
```

Use a fixed local preview port:

```bash
uv run zensical serve --dev-addr 127.0.0.1:8008
```

## Build

```bash
uv run zensical build --clean
```

The generated site is written to `site/`, which is intentionally ignored.

## Publish

GitHub Pages is deployed by:

```text
.github/workflows/docs.yml
```

GitHub repository settings should use:

```text
Settings -> Pages -> Build and deployment -> Source: GitHub Actions
```

## Source product repo

These docs currently track the local Arcessa product source at:

```text
/Users/pawan/enterprise-gateway
```

When product behavior changes, update the relevant docs page and run:

```bash
uv run zensical build --clean
```

## Contribution policy

Open issues for missing docs, unclear flows, stale commands, incorrect behavior, or broken navigation. See [CONTRIBUTING.md](CONTRIBUTING.md) before opening a PR.

## Security

Do not open public issues for security vulnerabilities or credential leaks. See [SECURITY.md](SECURITY.md).

## License

This documentation repository is licensed under the [MIT License](LICENSE).
