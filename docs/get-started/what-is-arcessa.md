---
title: What is Arcessa? - Governed Gateway for MCP Tools and A2A Agents
icon: lucide/compass
description: A plain-English introduction to Arcessa for first-time readers.
---

# What is Arcessa?

Arcessa is a control plane for AI tool access.

That sentence is accurate, but a first-time reader usually needs the more practical version:

> Arcessa is the place your AI clients and agents connect to before they touch company tools, APIs, data, or other agents.

Instead of giving every assistant direct credentials to every tool, you put Arcessa in the middle. Cursor, Claude Desktop, CI jobs, internal agents, and external partner systems call Arcessa. Arcessa then handles discovery, authentication, authorization, governance, safety checks, metering, routing, and audit.

## Table Of Contents

1. [The problem it solves](#the-problem-it-solves)
2. [Before and after](#before-and-after)
3. [The mental model](#the-mental-model)
4. [What can connect to it?](#what-can-connect-to-it)
5. [What can sit behind it?](#what-can-sit-behind-it)
6. [Vocabulary](#vocabulary)
7. [The first 30 minutes](#the-first-30-minutes)

## The problem it solves

AI tool access gets messy quickly.

One team adds a GitHub MCP server. Another adds a database tool. Someone else wraps internal APIs. A few agents begin calling each other over A2A. Developers use Cursor locally. Security asks who called what. Finance asks why model spend jumped. SRE asks whether a failed incident tool call was retried. Compliance asks for evidence.

Without a control plane, the answer is usually scattered across local config files, logs, API gateways, chat transcripts, and vendor dashboards.

Arcessa gives those flows one governed front door.

## Before and after

| Without Arcessa | With Arcessa |
|---|---|
| Every AI client stores its own tool secrets. | Clients use one Arcessa token or OAuth flow. |
| Tools are discovered by word of mouth. | Tools, resources, prompts, roots, gateways, and agents live in a catalog. |
| Authorization is bolted onto each tool. | Scopes, permissions, teams, orgs, and policies are enforced centrally. |
| Prompt injection and secret leakage are tool-specific problems. | Guardrails scan requests and responses across MCP and A2A flows. |
| Spend is hard to attribute. | Usage, tokens, cost, cache hits, and budgets are tracked by org/team/user/key/agent. |
| Audits require stitching logs together. | Every invocation produces a structured audit event and session trail. |
| Testing a tool requires a real client. | The Developer Console can browse, test, debug auth, save collections, and replay calls. |

## The mental model

``` mermaid
flowchart LR
  client[AI clients and agents] --> edge[Arcessa gateway]
  edge --> identity[Identity and tokens]
  edge --> mcp[MCP runtime]
  edge --> a2a[A2A gateway]
  mcp --> catalog[Inventory catalog]
  a2a --> catalog
  mcp --> policy[Governance]
  a2a --> policy
  policy --> decision{Allow, block, approve, redact, flag}
  decision --> upstream[Tools, APIs, MCP servers, agents]
  upstream --> audit[Audit, usage, sessions, SIEM]
```

Think of Arcessa as five layers working together:

| Layer | Plain-English job |
|---|---|
| Gateway | The public front door. It receives client traffic, applies edge checks, and routes internally. |
| Identity | Knows who the caller is, how they authenticated, what tenant they belong to, and what they can do. |
| Inventory | Knows what tools, resources, prompts, roots, gateways, and agents exist. |
| Governance | Decides whether a call is allowed, blocked, needs approval, should be redacted, or exceeds budget. |
| Observability | Records what happened so humans and systems can investigate later. |

## What can connect to it?

| Caller | Common flow |
|---|---|
| Cursor | Scoped Personal Access Token against `/mcp`. |
| Claude Desktop | Scoped Personal Access Token against `/mcp`. |
| OAuth-capable MCP client | OAuth 2.1 dynamic registration, PKCE, consent, refresh rotation. |
| CI job | Scoped API token with minimal scopes. |
| Internal service | Service token or external IdP bearer-token exchange. |
| Enterprise user | OIDC or SAML SSO. |
| HR/IT provisioning | SCIM 2.0 user provisioning. |
| A2A agent | Agent registration, card discovery, tasks, streaming, push. |

## What can sit behind it?

| Backend thing | How Arcessa treats it |
|---|---|
| MCP server | Proxied through the MCP runtime with protocol negotiation and audit. |
| REST API | Registered as a catalog tool and invoked as a governed tool. |
| Streamable HTTP endpoint | Handled as MCP front-door or upstream transport. |
| SSE endpoint | Supported for upstream streaming responses. |
| WebSocket upstream | Dispatched by URL scheme. |
| stdio MCP server | Optional, controlled local subprocess transport. |
| A2A agent | Registered in the agent gateway with cards, tasks, streaming, and push. |
| Upstream gateway | Federated discovery and health with loop guards. |

## Vocabulary

| Term | Meaning |
|---|---|
| MCP | Model Context Protocol, the protocol clients use to discover and call tools/resources/prompts. |
| A2A | Agent-to-agent protocol surface for discovering and invoking agents. |
| Tool | A callable capability exposed to an AI client. |
| Resource | Data exposed over MCP, often read-only. |
| Prompt | A reusable prompt template exposed over MCP. |
| Root | A filesystem or logical root advertised to MCP clients. |
| Gateway | A cataloged upstream MCP gateway or federation peer. |
| Policy | A rule that allows, blocks, or requires approval. |
| Guardrail | A content safety rule that redacts, blocks, or flags risky content. |
| Budget | A spend cap or warning threshold. |
| Session replay | A timeline view of tool/agent activity for investigation. |

## The first 30 minutes

1. Run the [Quickstart](quickstart.md).
2. Open the UI and complete the [First product tour](first-tour.md).
3. Create a PAT and [connect Cursor or Claude](../mcp/connect-clients.md).
4. Register one low-risk tool with [Register tools](../mcp/register-tools.md).
5. Add one policy and one guardrail with [Policies and approvals](../governance/policies.md).
6. Make a call and inspect [Audit, usage, and SIEM](../observability/audit-usage-siem.md).

!!! tip "A useful evaluation question"

    Do not only ask "can this proxy an MCP call?" Ask "can this explain who called what, why it was allowed, what it cost, whether sensitive data moved, and how to reproduce it?"
