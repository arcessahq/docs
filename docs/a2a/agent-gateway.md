---
title: A2A Agent Gateway - Agent Cards, Tasks, Streaming, Push, and Governance
description: Register and govern A2A agents through Arcessa with agent cards, task state, streaming, push callbacks, dialect handling, guardrails, and audit.
icon: lucide/bot
---

# A2A agent gateway

The A2A service lets teams register agents, serve discoverable agent cards, govern invocation, track tasks, receive push updates, and interoperate with real A2A SDK peers.

An agent is not just another URL. It can make decisions, call tools, stream progress, ask for more input, and produce artifacts. That makes agent access a governance problem, not just a routing problem.

## Table Of Contents

1. [What Arcessa adds to A2A](#what-arcessa-adds-to-a2a)
2. [Register an agent](#register-an-agent)
3. [Invoke an agent](#invoke-an-agent)
4. [Streaming invocation](#streaming-invocation)
5. [Task lifecycle](#task-lifecycle)
6. [Agent cards](#agent-cards)
7. [Protocol dialects](#protocol-dialects)
8. [Push callbacks](#push-callbacks)
9. [What to test for every agent](#what-to-test-for-every-agent)

## What Arcessa adds to A2A

| A2A concern | Arcessa behavior |
|---|---|
| Discovery | Registers agents and serves agent cards with tenant-aware redaction. |
| Trust | Fetches and retains published cards and verifies signatures when present. |
| Invocation | Routes governed `message/send` or version-specific calls to upstream agents. |
| Tasks | Stores local task state, event history, cancellation, and remote task ids. |
| Streaming | Relays upstream stream updates to callers over SSE. |
| Push | Accepts token-gated task push callbacks and can dispatch outbound webhooks. |
| Governance | Applies policies and guardrails to input, output, streams, and push callbacks. |
| Observability | Emits audit/usage events with tenant, task, token, cost, and guardrail context. |

## Register an agent

```bash
curl -s http://localhost:8080/api/v1/agents \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "support_agent",
    "slug": "support-agent",
    "description": "Handles support triage",
    "endpoint_url": "https://agents.example.com/a2a",
    "agent_type": "a2a",
    "visibility": "team",
    "team_id": "<team-id>"
  }' | jq
```

## Invoke an agent

```bash
curl -s http://localhost:8080/api/v1/agents/<agent-id>/invoke \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{
    "message": "Summarize open P1 incidents",
    "metadata": {
      "customer": "acme"
    }
  }' | jq
```

## Streaming invocation

```bash
curl -N http://localhost:8080/api/v1/agents/<agent-id>/invoke \
  -H "Authorization: Bearer $TOKEN" \
  -H "Accept: text/event-stream" \
  -H 'Content-Type: application/json' \
  -d '{"message":"Stream a triage plan"}'
```

## Task lifecycle

| State | Meaning |
|---|---|
| submitted | Arcessa accepted the task. |
| working | Upstream agent is processing. |
| input-required | Agent is waiting for more user input. |
| completed | Terminal success. |
| failed | Terminal failure. |
| canceled | Terminal cancellation. |

## Agent cards

Agent cards describe what an agent is and how to use it. Arcessa treats the upstream card as discovery metadata, not as a blindly trusted client-supplied payload.

Owner/admin view can include:

- endpoint URL
- security requirements
- full capabilities
- internal metadata

Non-owner view should redact:

- internal endpoint details
- secrets or credential hints
- security configuration that should not be broadly visible

Signed cards are verified when provided. A tampered card should fail registration or trust checks.

## Gateway-specific behavior

- Agent card metadata is fetched and retained at registration.
- Signed cards are verified when provided.
- Non-owner discovery redacts internal endpoint and security details.
- A2A requests are shaped per peer protocol version.
- Guardrails scan sync, streaming, and push-callback paths.
- Invocation events carry token, cost, guardrail, tenant, and task context.

## Protocol dialects

A2A implementations may differ by protocol version. Arcessa shapes outbound calls based on the agent card:

| Dialect | Example behavior |
|---|---|
| v0.3 | Slash-style methods such as `message/send`, `role: user`, `{kind:text}` parts. |
| v1 | `SendMessage`, `A2A-Version: 1.0`, `ROLE_USER`, `{text}` parts, `result.task`. |

This matters because an agent gateway must interoperate with real peers, not only its own fixtures.

## Push callbacks

For agent-to-gateway push, EGW registers a per-task callback and exposes a token-gated receiver:

```text
POST /a2a/push/{token}
```

Use this for real agent progress updates. It is distinct from EGW's own outbound webhook dispatcher for notifying external systems about agent events.

## What to test for every agent

| Scenario | Why |
|---|---|
| Happy path invoke | Basic shape and credential injection work. |
| Upstream `401` | Missing/wrong credentials do not look like success. |
| Upstream `500` | Task fails cleanly and is audited. |
| Streaming cancel | Local cancel aborts in-flight stream and federates remote cancel when possible. |
| `input-required` | State transition can be canceled or resumed correctly. |
| Bad push secret | Push receiver rejects invalid updates. |
| Timeout | Gateway degrades cleanly and task state is understandable. |
| Cross-org task id | Another tenant cannot read or cancel the task. |
| Guardrail block | Sensitive input/output is stopped. |
| Card redaction | Non-owner does not see endpoint/security internals. |
