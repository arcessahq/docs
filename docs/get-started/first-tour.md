---
icon: lucide/map
description: A guided product tour for first-time Arcessa users.
---

# First product tour

This tour is written for someone opening Arcessa for the first time. It explains what to click, what each area means, and what signal to look for.

## Start with the shape of the system

After running the [Quickstart](quickstart.md), sign in to:

```text
http://localhost:3000
```

The UI is split into operational areas:

| Area | What it answers |
|---|---|
| Catalog | What tools, resources, prompts, roots, and gateways exist? |
| Agents | What A2A agents are registered, callable, and discoverable? |
| Govern | What rules decide whether a call is allowed, blocked, approved, redacted, or capped? |
| Observability | What happened, who did it, what did it cost, and how do we investigate it? |
| Console | Can a developer discover, test, debug, save, and replay calls without leaving Arcessa? |
| Admin | Who can access Arcessa, which IdPs are configured, and what roles/tokens exist? |

## Tour path

### <span class="arcessa-step">1</span> Catalog

Open the Catalog section first. This is the inventory of capabilities you are making available to AI systems.

Look for:

- Tools: callable functions, REST endpoints, and MCP upstream calls.
- Resources: readable MCP resources.
- Prompts: reusable prompt templates.
- Roots: MCP roots exposed to clients.
- Gateways and servers: upstream MCP gateway or federation entries.
- Tags and visibility: ownership and discoverability metadata.

The important habit: create catalog entries through Arcessa, not by handing tool URLs and secrets directly to each client.

### <span class="arcessa-step">2</span> Access Tokens

Open Access Tokens. This is where developer-local flows become manageable.

Use it to create:

- Personal Access Tokens for Cursor, Claude Desktop, VS Code, curl, and local scripts.
- Admin API keys for automation.
- Narrow scoped keys for CI.

Good first token:

```text
mcp:invoke
catalog:read
```

That lets a local MCP client discover and invoke tools without giving it admin powers.

### <span class="arcessa-step">3</span> Console

Open Console. This is the "developer workbench" inside Arcessa.

Use it before debugging in a real client:

- Browse & Try: discover tools and invoke them interactively.
- Tester: test tools, resources, prompts, sampling, and elicitation flows.
- Auth Debugger: validate gateway/tool auth without exposing secrets.
- Collections: save calls and replay assertion suites.
- My Calls: see your own recent call trace.

If something fails in Cursor, first reproduce it in Console. That separates client config problems from gateway/runtime problems.

### <span class="arcessa-step">4</span> Govern

Open Govern after you have at least one tool.

Create one policy:

- `allow` for normal tools.
- `block` for dangerous operations.
- `require_approval` for production-impacting actions.

Create one guardrail:

- `secrets` detector with `block`.
- `pii` detector with `redact` or `block`.
- `prompt_injection` detector with `flag` or `block`.

Create one budget:

- Scope: team or user.
- Window: monthly.
- Warning threshold before hard cap.

The goal is to see `_meta.governance` on real calls, not just to create control-plane rows.

### <span class="arcessa-step">5</span> Observability

Make one tool call, then open:

- Audit
- Usage
- Sessions
- Metrics or SIEM if configured

Ask four questions:

1. Who called the tool?
2. Which tenant/workspace did the call belong to?
3. What decision did governance make?
4. What did the call cost or emit?

If those four answers are visible, the basic control-plane loop is working.

### <span class="arcessa-step">6</span> Identity

Open Authentication and Admin areas.

Review:

- Local bootstrap admin.
- OIDC providers.
- SAML configuration.
- SCIM token and org scoping.
- Roles and permissions.
- External bearer-token trust configuration.

For enterprise use, identity is not a side feature. It is what prevents "one useful tool" from becoming "everyone can call everything."

## What "done" looks like

| Signal | Expected result |
|---|---|
| Client can connect | Cursor or curl can call `/mcp` with a scoped token. |
| Catalog is populated | At least one tool/resource/prompt/agent is visible. |
| Governance is visible | Calls return `_meta.governance` or create approvals. |
| Audit exists | Invocation appears in audit and session views. |
| Secrets are hidden | No upstream credential appears in list/get/export/UI. |
| Tenant scope works | A non-admin sees only allowed org/team/private objects. |

!!! warning "Do not evaluate only the happy path"

    A production gateway earns trust on wrong tokens, wrong tenants, expired sessions, bad upstream schemas, retries, timeouts, revoked keys, and policy denials.
