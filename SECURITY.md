# Security Policy

## Reporting vulnerabilities

Do not report security vulnerabilities through public GitHub issues, discussions, or pull requests.

Report privately to the Arcessa maintainers with:

- A short description of the issue.
- Steps to reproduce.
- Impact and affected documentation or generated site path.
- Any relevant screenshots, logs, or proof-of-concept snippets.

## What counts as a security issue here

This repository is documentation, but security issues can still happen. Examples:

- Real credentials, API keys, tokens, private keys, or customer data in docs.
- Instructions that tell users to disable important security controls in production.
- Incorrect OAuth, SAML, SCIM, or token guidance that would cause unsafe deployments.
- Broken security metadata or misleading production-hardening guidance.
- Generated site metadata that leaks private URLs or internal repo paths.

## Safe documentation rules

- Use placeholders such as `<token>`, `<secret>`, and `api.example.com`.
- Do not paste real `.env` values.
- Do not include private hostnames, tenant names, or customer identifiers.
- Prefer least-privilege examples for scopes, roles, and tokens.
- Clearly separate local-dev shortcuts from production guidance.

## Dependency and workflow security

The docs build uses GitHub Actions and `uv`. Keep action versions and locked dependencies current. Dependabot is configured to raise update PRs for GitHub Actions and Python package metadata.
