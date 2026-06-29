---
icon: lucide/layout-dashboard
description: How developers use the Arcessa Console to discover, test, debug, and replay MCP/A2A calls.
---

# Developer Console

The Console is the fastest way to test Arcessa from a developer persona. It exists because "try it in Cursor" is a slow debugging loop: if a call fails, you do not know whether the problem is client config, token scopes, catalog visibility, governance, upstream auth, transport handling, or the upstream itself.

Use Console to prove the behavior inside Arcessa first.

## What the Console includes

| View | Purpose |
|---|---|
| Browse & Try | Discover visible tools and run a governed call. |
| Tester | Exercise tools, resources, prompts, sampling, completion, and elicitation flows. |
| Auth Debugger | Validate upstream auth handshakes and see redacted steps. |
| Collections | Save calls, replay them, and build lightweight assertion suites. |
| My Calls | Inspect your own recent call trace and governance metadata. |

## Browse & Try

Browse & Try is the fastest way to answer:

- Can this user see the tool?
- Does the token have the right scope?
- Does governance allow the call?
- Does the upstream return a usable result?
- Does audit record the call?

Typical flow:

1. Pick a tool from the visible catalog.
2. Fill generated fields from its schema.
3. Submit the call.
4. Inspect result body and `_meta.governance`.
5. Open Observability to confirm the event.

## Tester

Tester is broader than one tool call. It is for protocol coverage.

Use it to test:

| Protocol surface | Examples |
|---|---|
| Tools | `tools/list`, `tools/call`, schema-driven arguments. |
| Resources | `resources/list`, `resources/read`, templated resources. |
| Prompts | `prompts/list`, `prompts/get`, template variables. |
| Completion | Prompt/resource completion behavior. |
| Sampling | Sampling request/response path. |
| Elicitation | Interactive data collection flows. |

This is the right place to verify a new MCP or A2A integration before handing it to developers.

## Auth Debugger

Auth Debugger helps with upstream credentials and gateway auth. It should show enough to debug without leaking secrets.

Good debugger output includes:

- Which auth type was selected.
- Whether token minting or header injection happened.
- Which endpoint was contacted.
- Whether TLS/discovery succeeded.
- Redacted request/response metadata.
- Final status and error classification.

It must not include:

- Raw access tokens.
- Client secrets.
- Private keys.
- Upstream bearer headers.
- SIEM secrets.

## Collections

Collections are saved requests. They let a developer turn "I tried this once" into a reproducible check.

Use collections for:

- Regression checks before changing a tool.
- Demonstrating policy behavior.
- Replaying a set of common agent tasks.
- Capturing a minimal failing example for SRE or platform teams.

Example assertions:

| Assertion | Why it matters |
|---|---|
| Status is success | Basic integration health. |
| Governance decision is `allow` | Policy did what was expected. |
| Governance decision is `require_approval` | Risky call parked correctly. |
| Output does not contain sentinel secret | Redaction still works. |
| Cost is below expected ceiling | Pricing/cost attribution looks sane. |

## My Calls

My Calls is the developer's personal trace. It should answer:

- What did I call?
- Which request failed?
- Which governance decision applied?
- Was the result cached?
- Did a guardrail fire?
- Was the call recorded in audit?

This is the bridge between local client testing and platform observability.

## How to test a new integration

1. Register the tool or agent in Catalog/Agents.
2. Confirm it appears in Browse & Try.
3. Run a happy-path call.
4. Run a bad-schema call.
5. Run a timeout or failing upstream scenario if available.
6. Add one policy and prove the decision changes.
7. Add one guardrail and prove request/response scanning works.
8. Save the call in a collection.
9. Confirm audit/session/usage records exist.

!!! success "A complete integration is not just callable"

    A complete integration is discoverable, scoped, governed, guarded, observable, reproducible, and debuggable by someone who did not write it.
