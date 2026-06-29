---
title: Policies and Approvals in Arcessa
description: Use Arcessa policies to allow, block, or require approval for MCP tools and A2A agents with argument-aware conditions and auditable auto-resume.
icon: lucide/scale
---

# Policies and approvals

Policies decide whether an MCP tool, resource, prompt, or A2A agent call is allowed, blocked, or parked for human approval.

Governance is the reason Arcessa is more than a proxy. A proxy asks "where should this request go?" Governance asks "should this request happen at all, under this identity, with these arguments, right now?"

## Table Of Contents

1. [Policy effects](#policy-effects)
2. [Policy matching model](#policy-matching-model)
3. [Require approval for a tool](#require-approval-for-a-tool)
4. [Block by argument condition](#block-by-argument-condition)
5. [Good policy examples](#good-policy-examples)
6. [Approval flow](#approval-flow)
7. [Operational checklist](#operational-checklist)
8. [Approval quality bar](#approval-quality-bar)

## Policy effects

| Effect | Runtime behavior |
|---|---|
| `allow` | Continue invocation. |
| `block` | Return a governed denial. |
| `require_approval` | Create an approval, return pending state, and allow auto-resume after approval. |

## Policy matching model

Policies can match on:

- tool or agent name patterns
- priority
- argument conditions
- tenant/user/team context
- policy effect

High-priority explicit deny rules should be easy to reason about. Approval rules should be reserved for actions where human judgment is worth the delay.

## Require approval for a tool

```bash
curl -s http://localhost:8080/api/v1/policies \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "approve-prod-db",
    "effect": "require_approval",
    "tool_pattern": "prod_db_*",
    "priority": 100
  }' | jq
```

## Block by argument condition

```bash
curl -s http://localhost:8080/api/v1/policies \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "block-delete",
    "effect": "block",
    "tool_pattern": "*",
    "priority": 200,
    "conditions": {
      "path": "operation",
      "op": "eq",
      "value": "delete"
    }
  }' | jq
```

## Good policy examples

| Policy | Effect | Why |
|---|---|---|
| Block production deletes from AI tools | `block` | Some actions are too risky for autonomous execution. |
| Require approval for database export | `require_approval` | Humans should review data movement. |
| Allow low-risk read-only tools | `allow` | Avoid approval fatigue. |
| Block tools with `env=prod` unless team owns them | `block` | Prevent accidental cross-team production impact. |
| Require approval when `amount_usd > 1000` | `require_approval` | Argument-aware controls beat name-only controls. |

## Approval flow

``` mermaid
sequenceDiagram
  participant Client
  participant Runtime
  participant Governance
  participant Admin
  participant Upstream

  Client->>Runtime: tools/call
  Runtime->>Governance: evaluate
  Governance-->>Runtime: require_approval
  Runtime-->>Client: pending approval_id
  Admin->>Governance: approve
  Governance->>Runtime: internal execute approved call
  Runtime->>Upstream: invoke exact tool_id
  Runtime-->>Governance: result
```

## Operational checklist

- Prefer explicit high-priority block policies for dangerous operations.
- Use require-approval for production-impacting tools.
- Include team and org context in approval queues.
- Auto-resume must execute the exact approved `tool_id`, not a name-based lookup.
- Policies should have tests for allow, block, approval, and no-match paths.

## Approval quality bar

An approval entry should contain enough information for an admin to decide:

- caller identity
- org/team
- tool or agent id
- requested arguments
- matched policy
- creation time
- current state
- audit/session link when available

!!! warning "Approval is not a bypass"

    Approved calls still need the same post-approval execution safety: exact object id, guardrails, audit, and tenant context.
