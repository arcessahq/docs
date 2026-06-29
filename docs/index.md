---
icon: lucide/shield-check
---

# Arcessa documentation

<div class="arcessa-hero" markdown>

This is the technical documentation for Arcessa.

Use it to run the product, connect MCP clients, configure identity, register tools and agents, enforce governance, inspect audit trails, and operate production-like test environments.

[Start local quickstart](get-started/quickstart.md){ .md-button .md-button--primary }
[Tour the product](get-started/first-tour.md){ .md-button }
[See all capabilities](concepts/capability-map.md){ .md-button }
[Landing site](https://arcessahq.github.io/web/){ .md-button }

<div class="arcessa-pill-row" markdown>
<span class="arcessa-pill">MCP Gateway</span>
<span class="arcessa-pill">OAuth AS</span>
<span class="arcessa-pill">A2A Gateway</span>
<span class="arcessa-pill">SSO / SCIM</span>
<span class="arcessa-pill">Audit / SIEM</span>
</div>

</div>

``` mermaid
flowchart LR
  client[Cursor, Claude, CI, agents] --> gateway[Arcessa Gateway]
  gateway --> identity[Identity: OAuth, SSO, SCIM, tokens]
  gateway --> runtime[MCP Runtime]
  gateway --> a2a[A2A Gateway]
  runtime --> inventory[Inventory catalog]
  runtime --> governance[Governance policies and guardrails]
  a2a --> governance
  runtime --> upstream[MCP, REST, SSE, WS, stdio upstreams]
  a2a --> agents[A2A agents]
  runtime --> obs[Audit, usage, sessions]
  a2a --> obs
```

## Start here

- First-time technical reader: [What is Arcessa?](get-started/what-is-arcessa.md)
- Product/marketing overview: [Arcessa landing site](https://arcessahq.github.io/web/)
- New operator: [Quickstart](get-started/quickstart.md)
- Product evaluator: [Capability map](concepts/capability-map.md)
- Local developer: [Local development](get-started/local-dev.md)
- Cursor or Claude user: [Connect MCP clients](mcp/connect-clients.md)
- Developer testing tools: [Developer Console](console/developer-console.md)
- Platform admin: [OAuth and API tokens](identity/oauth.md)
- Security reviewer: [Production hardening](security/production-hardening.md)
- QA/SRE: [Testing overview](testing/overview.md)

## Core ideas

| Area | What Arcessa provides |
|---|---|
| MCP front door | One `/mcp` endpoint with Streamable HTTP, SSE, resumability, protocol negotiation, and upstream dispatch. |
| Identity | Local auth, scoped tokens, OAuth 2.1 authorization server, OIDC/SAML SSO, SCIM provisioning, and external IdP bearer-token exchange. |
| Governance | Policies, approvals, budgets, content guardrails, trust classification, and audit stamping on each invocation. |
| A2A | Agent registry, agent cards, governed invocations, task lifecycle, push callbacks, streaming, federation, and SDK interop tests. |
| Observability | ClickHouse audit and usage data, session replay, SIEM export, Prometheus metrics, and OpenTelemetry export. |
| Operations | Dockerized real-org stack, testcontainers integration tests, provider matrix, failure injection, load tests, and security scans. |

!!! tip "Use the docs as runnable playbooks"

    Most pages include exact commands against `localhost:8080` and assume the local quickstart stack unless the page explicitly says `real` or `production`.

## Choose your path

<div class="grid cards" markdown>

-   __Understand the product__

    ---

    Learn the plain-English model: clients, tools, agents, policies, identity, and audit.

    [:octicons-arrow-right-24: What is Arcessa?](get-started/what-is-arcessa.md)

-   __Run locally__

    ---

    Bring up infra, backend services, and the Next.js admin UI.

    [:octicons-arrow-right-24: Quickstart](get-started/quickstart.md)

-   __Connect an MCP client__

    ---

    Point Cursor, Claude Desktop, CI, or curl at the governed `/mcp` endpoint.

    [:octicons-arrow-right-24: Client setup](mcp/connect-clients.md)

-   __Try tools like a developer__

    ---

    Use Browse & Try, Tester, Auth Debugger, Collections, and My Calls.

    [:octicons-arrow-right-24: Developer Console](console/developer-console.md)

-   __Pick an auth flow__

    ---

    Use PATs, OAuth 2.1, SSO, SCIM, or external IdP bearer-token exchange.

    [:octicons-arrow-right-24: OAuth and tokens](identity/oauth.md)

-   __Validate production behavior__

    ---

    Run unit, integration, e2e, real-org, and provider matrix suites.

    [:octicons-arrow-right-24: Testing overview](testing/overview.md)

</div>

## What Arcessa covers

<div class="grid cards" markdown>

-   __Discovery__

    ---

    Catalog MCP tools, REST APIs, prompts, resources, roots, upstream gateways, and agents.

-   __Governance__

    ---

    Enforce allow, block, approval, guardrail, budget, trust, and tenant rules inline.

-   __Access__

    ---

    Support PATs, API keys, OAuth 2.1, OIDC, SAML, SCIM, and external IdP bearer tokens.

-   __Observability__

    ---

    Audit every invocation, track spend and tokens, replay sessions, export to SIEM.

-   __Agents__

    ---

    Register A2A agents, serve cards, route tasks, stream updates, and govern output.

-   __Production validation__

    ---

    Run Dockerized real-org suites, provider matrices, failure injection, scans, and load tests.

</div>
