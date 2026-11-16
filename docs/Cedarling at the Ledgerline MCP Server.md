# Cedarling at the Ledgerline MCP Server\


The Ledgerline MCP server asks a Cedarling sidecar before running each tool, so every agent call made with an ID-JAG-derived token is decided by signed Ledgerline rules and leaves a signed TRACE record that anyone can check.

## Decision and scope

Cedarling runs next to the Ledgerline MCP server and decides every tool call. The ID-JAG flow stays exactly as tested with Solo on 2026-10-10: agentgateway still obtains the ID-JAG, redeems it at `ledger-demo.gluu.info` and calls the MCP server with the 5-second access token.

Why at the MCP server and not in agentgateway: agentgateway's external authorizer is gRPC only (Envoy ext\_authz), and its docs do not say the MCP tool name or arguments are sent to it. Its built-in rules can see the tool name but not the arguments, and they leave no signed record. At the MCP server, Cedarling sees the token, the tool and its arguments, and Ledgerline owns its own rules, which fits the two-company story.

In scope:

- A Cedarling sidecar beside the Ledgerline MCP server, called before each tool runs.
- A Ledgerline policy store, written and released in Agama Lab, delivered to Cedarling by the Janssen Lock server on `ledger-demo`.
- One signed TRACE `AUTHORIZATION_DECISION` record per tool call, stored by that Lock and checkable on cedarling.dev.

Out of scope: changes to agentgateway, to the ID-JAG issuance on `sterling-idp` or the Keycloak broker, and gateway-side `CAPABILITY_INVOKED` records in the first phase (they are phase 2, see Risks and decisions).

## Components

`ledger-demo` runs Janssen 2.6.1, which already includes everything this design needs on the server side: Lock TRACE ingestion, search and gRPC endpoints, the Cedarling TRACE emitter, and policy-store spec v2.

| Component | Owner | Runs where | Role in this design |
| --- | --- | --- | --- |
| `ledger-demo.gluu.info` (Janssen 2.6.1) | Gluu (Zico) | Existing server | Redeems the ID-JAG, issues the 5 s JWT access token, and hosts the Janssen Lock server |
| Janssen Lock server (`jans-lock`, part of 2.6.1) | Gluu | On `ledger-demo` | Serves the signed policy store to Cedarling, registers Cedarling's producer key, verifies and stores TRACE records |
| Cedarling sidecar (`cedarling-flask-sidecar`, Cedarling 2.6.10) | Gluu | Next to the MCP server | Validates the access token against `ledger-demo`'s keys, decides each tool call, signs a TRACE record |
| Ledgerline MCP server | Solo lab (kmcp) or Gluu | Today in Solo's lab | Calls the sidecar before each tool; runs the tool only on ALLOW |
| agentgateway | Solo | Solo lab | Unchanged: ID-JAG exchange and redemption, forwards the tool call with the access token |
| Agama Lab Policy Designer | Gluu | cloud.gluu.org | Authors and releases the Ledgerline policy store (Sigstore-signed `.cjar`) |
| cedarling.dev proof view | Gluu | Public site | Checks a downloaded proof bundle in the browser |

The sidecar is matched to `ledger-demo` and runs Cedarling 2.6.10. The Agama Lab sandbox pins 2.5.21, so a store must load in both before it is released.

## Call flow

Steps 1 to 4 are the existing ID-JAG flow; steps 5 and 6 are new.

&#91;embedded content: call flow · 6 steps, 2 companies\]

1. Bob's agent calls a Ledgerline tool through agentgateway.
2. agentgateway exchanges Bob's ID token for an ID-JAG at Sterling & Vance's IdP.
3. agentgateway redeems the ID-JAG at `ledger-demo` and gets a 5-second JWT access token.
4. agentgateway calls the MCP server with that token and a trace id header.
5. Before running the tool, the MCP server asks the Cedarling sidecar, sending the access token, the tool name as the action, a Jans::Application resource and the context, with fraud\_indicators set when it detects a prompt injection. The tool runs only on ALLOW; DENY returns an MCP error.
6. Cedarling signs an `AUTHORIZATION_DECISION` record and sends it to the Janssen Lock server, which verifies and stores it. Lock also delivers the signed policy store to Cedarling.

Headers the MCP server sets on every sidecar call; without the first two, the sidecar writes no TRACE record: `Audit-Trace-ID` (the gateway's request id, or `exec-<uuid>` if none arrives), `Audit-Execution-Authority` (`ledgerline`), `Audit-Workload-ID` (the MCP server's name) and `Audit-Workload-Instance-ID`.

## Policy store

One store, `ledgerline_research_mcp`, written in Agama Lab and kept in [mzico/ledgerline-policies](https://github.com/mzico/ledgerline-policies) (branch `agama-lab-policy-designer`). v0.1.2, with the fixes from [PR #1](https://github.com/mzico/ledgerline-policies/pull/1), is released and served by `ledger-demo`'s Lock. Cedarling runs in token mode (`authorize_multi_issuer`): identity comes only from the `ledger-demo` access token, so no rule constrains `principal` in its head.

Trusted issuer: named `Jans`, endpoint `https://ledger-demo.gluu.info/.well-known/openid-configuration`, access token mapped to `Jans::Access_token`. Its claims reach the rules as `context.tokens.jans_access_token`. A first release with the issuer named `Ledgerline` (v0.1.0) did not load in Cedarling 2.6.10.

| Rule | Kind | Condition | What it prevents |
| --- | --- | --- | --- |
| `read-research` | permit | action `Search` or `Read`, token `scp` contains `research:read`, token `aud` contains `ledgerline-research` | Calls with a token meant for another service |
| `block-export` | forbid | action `Share` | Research leaving Ledgerline through an agent |
| `block-suspicious` | forbid | `context.fraud_indicators` contains `prompt_injection` (set by the MCP server) | Acting on a hijacked request |

Shape of a request:

- Actions `Search`, `Read` and `Share` are placeholders until Solo sends the real MCP tool names.
- The resource is a `Jans::Application` with `app_id`, `name` and `url`; the MCP server sends it, e.g. `app_id` = the tool name, `url` = the MCP endpoint.
- Token claims: `scp` (array) and `aud` (array) are read by the rules; `scope` stays a plain string.
- The schema's `TokensContext` declares access, ID and userinfo tokens, so the same store also runs on the Agama Lab sandbox, which signs test users in with all three.

## Evidence

Every tool call leaves one signed `AUTHORIZATION_DECISION` record in the Janssen Lock server on `ledger-demo`, retrievable by trace id or by policy release. This is the same mechanism proven end to end on the Agama Lab sandbox on 2026-10-28.

What a record signs: the action (tool), the resource type, ALLOW or DENY, references to the tokens used (issuer and token id only, never the token), the policy store id, version and `bundle_hash`, and the trace id and execution authority. It does not sign the tool arguments, the rule that decided, or the person's name.

How it is set up:

- Cedarling gets its own Ed25519 producer key, registered at Lock with `authorized_event_kinds` = `AUTHORIZATION_DECISION`.
- Lock binds the sidecar's OAuth client to a `ledgerline` evidence domain; records are signed with `trace.write` and read with `trace.readonly`.
- `bundle_hash` is the SHA-256 of the exact `.cjar` Lock served, so a record names the policy release it was decided under.

How someone checks it: open Agama Lab's Evidence panel in "Another Lock" mode against `ledger-demo` with a `trace.readonly` token, load the record by trace id, and verify it on [cedarling.dev/playground/proof](https://cedarling.dev/playground/proof). A valid signature proves which key signed the record and that it was not changed; it does not prove the tool did what the decision allowed.

Agama Lab's "Download proof bundle" button only works for sandbox evidence today, because it needs the producer's public key, which only the sandbox keeps. For `ledger-demo`, the bundle is assembled from Lock's record, the key listed by Lock's `GET /audit/trace/admin/producer-keys` and the released `.cjar` by tools/make\_proof\_bundle.py in mzico/ledgerline-policies.

## Agama Lab

Agama Lab is where Ledgerline's rules are written, signed and rehearsed; it is not in the call path.

1. Author the `ledgerline-research-mcp` store in the Policy Designer, with the rules above.
2. Rehearse on the sandbox: Test on sandbox, sign in as a test user and Run decision, with a test token and context shaped like `ledger-demo`'s. Note the sandbox sidecar pins cedarling-python 2.5.21 and needs one policy per file; `ledger-demo` runs 2.6.1.
3. Release: a Sigstore-signed `.cjar` as a GitHub release asset.
4. Deploy: point `ledger-demo`'s Lock at the release URL, the way the sandbox deploy does, so Lock serves it to Cedarling.
5. Inspect: the Evidence panel in "Another Lock" mode reads `ledger-demo`'s records.

## Work items

Status on Nov 16, 2026: the main path works on `ledger-demo`. With a real token and the real Flask sidecar, Search is allowed, Share is denied, Read with `prompt_injection` is denied, and Lock stored the signed records with valid signatures. Store v0.1.2 is live, and a signed Search decision made under it verifies on cedarling.dev.

| # | Item | Status |
| --- | --- | --- |
| 1 | Token claims | Done: `scp` array added; `scope` stays a string |
| 2 | Lock setup | Done: sidecar client, `ledgerline` evidence domain, `FIRST_OBSERVED` chain mode, signing key registered |
| 3, 4 | Store released and served | Done: v0.1.2 released ([PR #1](https://github.com/mzico/ledgerline-policies/pull/1) merged) and served; hash matches the release |
| 5 | Sidecar against Lock | Done with a pasted Lock token |
| 6 | MCP server pre-tool check | Open: send `Jans::Application`, `fraud_indicators` and both audit headers |
| 7 | Gateway trace id | Asked Nick; not blocking, the MCP server falls back to `exec-<uuid>` |
| 8 | Full ID-JAG flow on the rebuilt server | Open |
| 9 | Proof bundle for ledger-demo | Done: tools/make\_proof\_bundle.py (ledgerline-policies PR #2, merged); the v0.1.2 bundle's signature and .cjar hash verify |
| 12 | Re-test and release v0.1.2 | Done: aud and scp map on 2.6.10; Search allowed under v0.1.2 |
| 13 | Sandbox login | Intermittent: passed on retry; reopen with jans-auth logs if it recurs |
| 15 | cedarling.dev sample, proof and tutorial | Draft in GluuFederation/websites#83 with the v0.1.2 proof; waits on 8 and the real tool names |

| # | Work item | Owner | Depends on |
| --- | --- | --- | --- |
| 1 | Emit `scope` as an array (and add `email_verified` if the rule stays) in the `ledger-demo` access-token script; confirm with a live token | Zico | none |
| 2 | Enable the Janssen Lock server's policy and TRACE endpoints on `ledger-demo`; create the sidecar client with `policy.readonly` and `trace.write`, and a `ledgerline` evidence-domain binding | Zico | none |
| 3 | Author and rehearse `ledgerline-research-mcp` in Agama Lab; release it signed | Gluu | 1 (claim shapes) |
| 4 | Deploy the release to `ledger-demo`'s Lock | Gluu | 2, 3 |
| 5 | Run the Cedarling sidecar 2.6.1 next to the MCP server: Lock bootstrap, `ledger-demo` as trusted issuer, a producer key registered at Lock | Gluu with Solo | 2, 4 |
| 6 | Add the pre-tool check to the MCP server: call the sidecar, run the tool only on ALLOW, set the audit headers | Solo (kmcp server) or Gluu | 5 |
| 7 | Confirm whether agentgateway forwards a trace id header to the MCP server | Solo | none |
| 8 | End-to-end run: Bob's agent calls a tool, Lock stores a record, the record verifies on cedarling.dev | Solo, Zico | 6 |
| 9 | Proof bundle for a non-sandbox Lock: assemble record, producer key and `.cjar` (a small script, or an Agama Lab follow-up) | Gluu | 8 |
| 10 | Phase 2: agentgateway signs a CAPABILITY\_INVOKED record per forwarded tool call, with its own producer key registered at ledger-demo's Lock, same trace id and execution authority as the decision | Solo, with Gluu on Lock setup | 7, 8 |
| 11 | Phase 2: show the decision and the invocation side by side for one trace id in Agama Lab's Evidence panel and the cedarling.dev proof view | Gluu | 10 |
| 12 | Re-run the sidecar test with PR #1, then release v0.1.2 (or only its first commit if aud/scp do not map to attributes on 2.6.10) | Zico | none |
| 13 | Fix the sandbox test-user login (invalid\_authentication\_method, correlation id 8c2b1319-a74f-49f7-816b-7ffb7f49b9df) so the store can be rehearsed | Gluu | jans-auth log for that id |
| 14 | Replace the pasted Lock token with one that renews: client credentials from the sidecar's client, or registration if Cedarling 2.6.10 supports it | Gluu | none |
| 15 | Publish the Ledgerline Playground sample, recorded proof and tutorial on cedarling.dev; confirm the recorded call went through agentgateway, and swap in the real tool names | Gluu | 8, 9, Solo's tool names |

## Risks and decisions

| Risk or question | Effect | Mitigation |
| --- | --- | --- |
| The 5-second access token expires before Cedarling validates it | DENY on a slow call | Validate first thing in the request; keep clocks in sync (NTP), as the interop already requires |
| Token status-list check adds a fetch to `ledger-demo` per decision | Latency inside the 5-second window | Confirm `ledger-demo` serves `/jans-auth/restv1/status_list`; Cedarling caches it |
| `scope` stays a string | `read-research` never matches | Resolved: rules read the scp array |
| Real MCP tool names and argument shapes unknown | Rules and actions change | Get the tool list from Solo before work item 3 |
| Sandbox (2.5.21) and `ledger-demo` (2.6.1) differ in store format | A store that passes rehearsal fails on 2.6.1 | Rehearse a final run against `ledger-demo` before the event |
| agentgateway may not forward a trace id | Gateway log and TRACE record cannot be joined | Fall back to `exec-<uuid>` from the MCP server; ask Solo (work item 7) |

Decided: phase 2 adds gateway-side `CAPABILITY_INVOKED` records. agentgateway, as the enforcement point, signs one record per tool call it forwards, with its own producer key registered at `ledger-demo`'s Lock and the same trace id and execution authority as Cedarling's decision. An auditor then reads both records for one trace id from Lock: what was decided, and what was actually invoked. This depends on Solo adding a TRACE producer to agentgateway (work item 10).

Decided on Nov 16, 2026:

- `require-verified-email` is dropped for phase 1. Email verification is Sterling & Vance's fact, already checked at their IdP before it issues the ID-JAG.
- `fraud_indicators` replaces `context.suspicious`; no schema change is needed.
- Placeholder actions `Search`, `Read` and `Share` stay until Solo sends the real tool names.
- The MCP server, not agentgateway, sets `Audit-Trace-ID` and `Audit-Execution-Authority`. A forwarded gateway request id is used as the trace id when present.
