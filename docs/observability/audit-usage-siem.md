---
icon: lucide/activity
---

# Audit, usage, and SIEM

Every invocable path should emit a unified invocation event. Observability consumes events and builds audit trails, usage rollups, sessions, and SIEM exports.

Observability is not only dashboards. It is the evidence layer for security, SRE, finance, compliance, and developers debugging tool behavior.

## Event sources

| Source | Event examples |
|---|---|
| MCP runtime | `tools/call`, `resources/read`, `prompts/get`, sampling, elicitation. |
| A2A gateway | Agent invoke, streaming updates, push callback activity. |
| Governance | Approval resume and decision metadata. |
| Gateway | Budget denial and edge metrics. |

## What an invocation event should explain

| Field | Why it matters |
|---|---|
| Actor | Who or what made the call. |
| Token/key id | Which credential was used. |
| Org/team/user | Tenant and workspace attribution. |
| Tool/agent/resource/prompt | What capability was touched. |
| Decision | Allow, block, approval, guardrail, cache. |
| Latency | Runtime and upstream performance. |
| Tokens/cost | Spend attribution and budget enforcement. |
| Error | Debuggability and incident response. |
| Session id | Ability to reconstruct a chain of actions. |

## Audit query

```bash
curl -s 'http://localhost:8080/api/v1/audit?limit=20' \
  -H "Authorization: Bearer $TOKEN" | jq
```

## Usage summary

```bash
curl -s 'http://localhost:8080/api/v1/usage/summary?window=24h' \
  -H "Authorization: Bearer $TOKEN" | jq
```

## Session replay

Session replay is the "what happened in order?" view.

Use it when:

- an agent made several tool calls and one failed
- a policy approval resumed a call later
- a user claims a tool returned wrong data
- security needs to understand data movement
- SRE needs to correlate latency and errors

Good session data should show the call sequence, decisions, errors, guardrail activity, and related audit records.

## SIEM destination

Create a webhook SIEM sink:

```bash
curl -s http://localhost:8080/api/v1/siem/destinations \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "security-webhook",
    "type": "webhook",
    "url": "https://siem.example.com/arcessa",
    "format": "json",
    "secret": "shared-webhook-secret"
  }' | jq
```

## SIEM formats

| Format | Use when |
|---|---|
| JSON | Default, easiest to parse and preserve full context. |
| CEF | Security teams standardize on ArcSight-compatible fields. |
| LEEF | QRadar-style ingestion. |
| Splunk HEC | Direct Splunk event ingestion. |
| Datadog | Datadog log/event pipelines. |
| Syslog | Traditional infrastructure logging environments. |

## Metrics

Every service exposes:

```text
GET /metrics
```

Key counters include:

- `http_server_requests_total`
- `mcp_invocations_total`
- `mcp_guardrail_actions_total`
- `mcp_cache_hits_total`
- `mcp_cancellations_total`
- `gateway_budget_denied_total`

## Production checks

- ClickHouse response reads must be bounded.
- Audit reads must be scoped to the caller's org.
- Secret sentinel values must not appear in UI, API, export, or support bundle.
- Dropped NATS events should be logged.
- SIEM delivery should retry and expose failures clearly.

## Questions observability should answer

| Persona | Question |
|---|---|
| Developer | Why did my tool call fail? |
| Admin | Who approved this production action? |
| Security | Did a prompt-injection attempt reach an upstream? |
| Finance | Which team drove spend this week? |
| SRE | Did Redis/NATS/ClickHouse outages degrade safely? |
| Compliance | Can we produce an audit trail for sensitive calls? |
