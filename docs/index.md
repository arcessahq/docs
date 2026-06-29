---
title: Arcessa Documentation - MCP and A2A Gateway Control Plane
description: Technical documentation for Arcessa, the governed control plane for MCP tools and A2A agents with OAuth, SSO, SCIM, policies, guardrails, budgets, audit, and production testing.
icon: lucide/shield-check
---

# Arcessa Documentation

Welcome to the documentation for **Arcessa** - a governed control plane for MCP tools and A2A agents.

Use these docs to run Arcessa, connect AI clients, register tools and agents, configure identity, enforce governance, inspect audit trails, and operate production-like test environments.

## Table Of Contents

1. [What is Arcessa?](get-started/what-is-arcessa.md) - A plain-English introduction for first-time technical readers
2. [Quickstart](get-started/quickstart.md) - Run Arcessa locally and make your first governed MCP call
3. [First Product Tour](get-started/first-tour.md) - What to click in the UI and what each area means
4. [Capability Map](concepts/capability-map.md) - Complete product surface across MCP, A2A, identity, governance, observability, and operations
5. [Request Lifecycle](concepts/request-lifecycle.md) - How MCP and A2A calls move through the gateway
6. [Connect MCP Clients](mcp/connect-clients.md) - Cursor, Claude Desktop, CI, OAuth-capable clients, and static Bearer clients
7. [Register Tools](mcp/register-tools.md) - Add REST, MCP, WebSocket, SSE, and stdio-backed tools
8. [A2A Agent Gateway](a2a/agent-gateway.md) - Register agents, serve cards, invoke tasks, stream updates, and receive push callbacks
9. [OAuth and Tokens](identity/oauth.md) - PATs, API keys, OAuth 2.1, DCR, PKCE, resource-bound tokens, and external IdP bearer exchange
10. [Policies and Approvals](governance/policies.md) - Allow, block, require approval, argument conditions, and auto-resume
11. [Audit, Usage, and SIEM](observability/audit-usage-siem.md) - Invocation events, usage, cost, session replay, metrics, and SIEM export
12. [Testing Overview](testing/overview.md) - Unit, integration, e2e, real stack, provider matrix, load, failure, and security testing

## Introduction

Arcessa sits between AI clients and the tools, APIs, data, and agents they want to call. Instead of giving every IDE, assistant, CI job, or autonomous agent direct credentials to every backend, you point them at one governed endpoint.

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

That endpoint becomes the place where the platform team can answer:

- What tools and agents exist?
- Who can discover them?
- Which client, user, token, team, or org made a call?
- Was the call allowed, blocked, approved, redacted, cached, or rate-limited?
- What did it cost?
- What happened when the upstream failed?
- Can security and SRE reconstruct the session later?

## At A Glance

<div class="arcessa-glance" markdown>
<div markdown>
<strong>One endpoint</strong>
<span>Point Cursor, Claude, CI, and agents at `/mcp` instead of scattering tool URLs and secrets.</span>
</div>
<div markdown>
<strong>Inline governance</strong>
<span>Apply auth, scopes, tenant visibility, policies, approvals, guardrails, budgets, and caching before calls reach upstreams.</span>
</div>
<div markdown>
<strong>Complete evidence</strong>
<span>Record audit, usage, cost, sessions, SIEM events, metrics, and failure context for every invocable path.</span>
</div>
</div>

## Key Features

- **MCP front door:** one `/mcp` Streamable HTTP endpoint for Cursor, Claude Desktop, CI, and internal agents.
- **Full MCP surface:** tools, resources, prompts, roots, completion, sampling, elicitation, subscriptions, pagination, progress, and resumable streams.
- **A2A gateway:** agent registry, signed agent cards, task lifecycle, streaming, push callbacks, protocol dialect handling, and federation.
- **Identity:** PATs, scoped API keys, OAuth 2.1 authorization server, OIDC, SAML, SCIM, multi-IdP, and external IdP bearer-token exchange.
- **Governance:** allow, block, require approval, per-argument policy conditions, approvals, and auto-resume.
- **Safety:** guardrails for PII, secrets, prompt injection, request and response scanning, redact/block/flag actions.
- **Cost controls:** token/cost attribution, response caching, usage rollups, and budgets by org, team, user, agent, or key.
- **Observability:** audit trail, usage, session replay, SIEM export, Prometheus metrics, and OpenTelemetry.
- **Production validation:** Docker real-org stack, provider matrix, official SDK/TCK interop, failure injection, load, chaos, and security scans.

## Quick Example

Run Arcessa:

```bash
cd /Users/pawan/enterprise-gateway
cp .env.example .env
make infra-up
make dev
```

Get a token:

```bash
TOKEN=$(curl -s http://localhost:8080/api/v1/auth/login \
  -H 'Content-Type: application/json' \
  -d '{"email":"admin@local.dev","password":"changeme123"}' \
  | jq -r .token)
```

List MCP tools through the governed front door:

```bash
curl -s http://localhost:8080/mcp \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}' | jq
```

Connect Cursor:

```json
{
  "mcpServers": {
    "arcessa": {
      "url": "http://localhost:8080/mcp",
      "headers": {
        "Authorization": "Bearer ${env:ARCESSA_TOKEN}"
      }
    }
  }
}
```

## Common Workflows

| Workflow | Start here |
|---|---|
| Understand the product | [What is Arcessa?](get-started/what-is-arcessa.md) |
| Run locally | [Quickstart](get-started/quickstart.md) |
| Click through the UI | [First Product Tour](get-started/first-tour.md) |
| Connect Cursor or Claude | [Connect MCP Clients](mcp/connect-clients.md) |
| Add a tool | [Register Tools](mcp/register-tools.md) |
| Add an agent | [A2A Agent Gateway](a2a/agent-gateway.md) |
| Choose an auth flow | [OAuth and Tokens](identity/oauth.md) |
| Configure SSO or provisioning | [SSO and SCIM](identity/sso-scim.md) |
| Add policy and approvals | [Policies and Approvals](governance/policies.md) |
| Add guardrails and budgets | [Guardrails and Budgets](governance/guardrails.md) |
| Debug a developer flow | [Developer Console](console/developer-console.md) |
| Prove production posture | [Testing Overview](testing/overview.md) |

## Links

- **Landing site:** [arcessahq.github.io/web](https://arcessahq.github.io/web/)
- **Docs source:** [github.com/arcessahq/docs](https://github.com/arcessahq/docs)
- **Agent index:** [llms.txt](llms.txt)
- **Sitemap:** [sitemap.xml](sitemap.xml)

## License

The Arcessa documentation repository is licensed under the [MIT License](https://github.com/arcessahq/docs/blob/main/LICENSE).
