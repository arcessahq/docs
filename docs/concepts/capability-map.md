---
title: Arcessa Capability Map - MCP, A2A, Identity, Governance, Observability, and Operations
icon: lucide/list-checks
description: A product-wide map of what Arcessa provides and where each capability fits.
---

# Capability map

This page is the broad map. Use it when you want to understand everything Arcessa provides without reading the entire codebase or every reference page.

## Table Of Contents

1. [Control-plane areas](#control-plane-areas)
2. [MCP capabilities](#mcp-capabilities)
3. [Identity capabilities](#identity-capabilities)
4. [Governance capabilities](#governance-capabilities)
5. [Operations capabilities](#operations-capabilities)
6. [What to read next](#what-to-read-next)

## Control-plane areas

| Area | What Arcessa provides | Why it matters |
|---|---|---|
| MCP front door | One `/mcp` endpoint with Streamable HTTP, JSON-RPC, SSE, protocol negotiation, resumability, progress, and pagination. | AI clients get one stable endpoint instead of a sprawl of direct tool URLs. |
| Catalog | Tools, resources, prompts, roots, servers, gateways, tags, import/export, federation. | Teams can see what exists and control how it is exposed. |
| Identity | Local login, JWTs, roles, teams, custom roles, scoped API tokens, PATs. | Human and machine access can be separated and minimized. |
| OAuth AS | Dynamic client registration, PKCE auth code, refresh rotation, resource-bound audiences, protected-resource metadata. | Standards-capable MCP clients can do the full OAuth dance. |
| Enterprise SSO | OIDC, SAML, multi-IdP, access/ID/userinfo role mapping, SCIM provisioning. | Enterprises can use existing identity infrastructure instead of local-only accounts. |
| External IdP bearer exchange | Trust an enterprise JWKS and exchange Okta/Entra/Keycloak-style access tokens into Arcessa tokens. | Tools that cannot do Arcessa OAuth can still use enterprise-issued credentials. |
| Governance | Policies with allow/block/require-approval, argument conditions, priority, approval queue, auto-resume. | Risky calls can be denied or parked without breaking normal tools. |
| Guardrails | PII, secrets, prompt-injection detectors with redact/block/flag on request and response paths. | Content safety is enforced at the gateway instead of scattered per tool. |
| Budgets | Spend caps and warnings by org, team, user, agent, or key. | AI cost becomes governable and attributable. |
| Trust and data flow | Response classification and untrusted-context propagation. | Tool outputs can be treated according to trust level. |
| Response caching | Exact and optional semantic cache, per-tool TTL, cache-hit audit. | Repeated calls can be cheaper and faster without hiding usage. |
| A2A gateway | Agent registry, signed cards, redaction, tasks, streaming, push, federation, SDK interop. | Agents become governed first-class capabilities, not hidden side channels. |
| Observability | Audit, usage, tokens, cost, sessions, search, SIEM, metrics, OpenTelemetry. | Operators can answer who did what and why. |
| Edge operations | CORS, rate limiting, token revocation, header handling, proxying, reverse-proxy tunnel. | Public exposure is centralized and measurable. |
| Production validation | Unit, integration, e2e, real stack, provider matrix, security scan, load, chaos, Toxiproxy. | Changes can be tested against real behavior, not just mocks. |

## MCP capabilities

| Capability | Details |
|---|---|
| `initialize` | Protocol-version negotiation and capability advertisement. |
| `tools/list` | Tenant-aware discovery, optional pagination, group-scoped tokens. |
| `tools/call` | Governance, guardrails, cache, upstream dispatch, cost, audit. |
| `resources/list` and `resources/read` | Resource discovery and governed reads. |
| `resources/templates/list` | Template-aware resource discovery. |
| `prompts/list` and `prompts/get` | Prompt discovery, template completion, governance and audit. |
| `roots/list` | Tenant-scoped roots. |
| `completion/complete` | Prompt/resource template completion. |
| `sampling` | Controlled sampling surface. |
| `elicitation` | Client-interactive elicitation flow. |
| Subscriptions | Resource subscription behavior and streaming. |
| Resumability | SSE event ids, replay buffers, `Last-Event-ID`, owner-bound replay. |

## Identity capabilities

| Capability | Details |
|---|---|
| Local auth | Email/password, bootstrap admin, argon2id passwords, lockout controls. |
| RBAC | Built-in viewer/editor/admin and custom roles. |
| Fine-grained permissions | `tools.write`, `agents.write`, `policies.write`, `roles.manage`, and related gates. |
| Token scopes | Route-level scopes such as `mcp:invoke`, `catalog:read`, `agents:read`, `observe:read`. |
| PATs | Self-serve developer tokens with lifetime and escalation limits. |
| API keys | Admin-created scoped tokens for automation. |
| OAuth 2.1 | DCR, PKCE, auth code, refresh rotation, audience binding. |
| OIDC | Environment and runtime providers, role/team claim mapping. |
| SAML | SP configuration, IdP metadata, signed assertion validation. |
| SCIM | User CRUD/filter/PATCH/PUT, pagination, service provider config and schemas. |

## Governance capabilities

| Capability | What to test |
|---|---|
| Allow policy | Matching tool/agent proceeds and stamps decision. |
| Block policy | Matching call fails predictably and is audited. |
| Require approval | Call parks, approval is visible, auto-resume executes the approved object. |
| Argument conditions | Policy matches specific request content, not just names. |
| Guardrail request scan | Sensitive input is redacted, blocked, or flagged. |
| Guardrail response scan | Sensitive output is redacted, blocked, or flagged. |
| Budget warning | Warning header/event appears before hard cap. |
| Budget block | Gateway returns `402` when cap is exceeded. |
| Tenant scoping | Non-admin cannot see another org's approvals or catalog objects. |

## Operations capabilities

| Capability | Where to look |
|---|---|
| Metrics | `GET /metrics` on each service. |
| Traces/logs | OpenTelemetry endpoint config. |
| SIEM | Governance/observability SIEM destinations and delivery pipeline. |
| Support bundle | Gateway admin support bundle endpoint. |
| Real stack | `make real-up`, `make real-test`, `make matrix-all`. |
| Load | k6 scenarios for normal, spike, rate-limit, volume, and soak. |
| Chaos | NATS/Redis/service restarts and recovery checks. |
| Security scans | gitleaks, Trivy SBOM/image scans, ZAP HTTP checks. |

## What to read next

<div class="grid cards" markdown>

-   __Run it__

    [:octicons-arrow-right-24: Quickstart](../get-started/quickstart.md)

-   __Follow a call__

    [:octicons-arrow-right-24: Request lifecycle](request-lifecycle.md)

-   __Test like a developer__

    [:octicons-arrow-right-24: Developer Console](../console/developer-console.md)

-   __Harden it__

    [:octicons-arrow-right-24: Production hardening](../security/production-hardening.md)

</div>
