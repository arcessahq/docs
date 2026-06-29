---
icon: lucide/shield-alert
---

# Guardrails and budgets

Guardrails inspect request and response content. Budgets cap spend at org, team, user, agent, or key level.

## Guardrail detectors

| Detector | Finds |
|---|---|
| `pii` | Email, SSN, credit card, phone, IP-like data. |
| `secrets` | AWS keys, GitHub tokens, private keys, high-entropy strings. |
| `prompt_injection` | Common instruction override and data exfiltration patterns. |

## Actions

| Action | Behavior |
|---|---|
| `redact` | Mutates matched content before continuing. |
| `block` | Stops the invocation. |
| `flag` | Allows but stamps audit and metadata. |

## Create a guardrail

```bash
curl -s http://localhost:8080/api/v1/guardrails \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "block-secrets",
    "detectors": ["secrets", "pii"],
    "action": "block",
    "target": "request",
    "tool_pattern": "*"
  }' | jq
```

## Create a budget

```bash
curl -s http://localhost:8080/api/v1/budgets \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{
    "scope_type": "team",
    "subject": "<team-id>",
    "window": "monthly",
    "limit_usd": 500,
    "warn_usd": 400,
    "action": "block"
  }' | jq
```

## Runtime behavior

- Budget checks happen at the gateway before expensive invocations.
- Spend increments asynchronously from invocation events.
- `X-Egw-Budget-Warning` warns near threshold.
- Hard cap returns `402`.
- Cache hits carry `cached=true` and should not increase spend.

## Validation tests

- PII in request is blocked or redacted.
- Secret in response is blocked or redacted.
- Prompt-injection detector flags risky text.
- Budget warning header appears before cap.
- Budget hard cap returns `402`.
- Audit event carries guardrail and spend fields.
