# Connecting to Gluu for MCP / ID-JAG Interoperability Testing: Partner Guide

Last updated 2026-10-12 · Author: Mohib Zico, Gluu Inc.

## About this guide

This guide tells you what to prepare, what to send Gluu, what you will receive and how Gluu is configured, so your product can be tested with Gluu for AI-agent authorization using the IETF ID-JAG grant, MCP and CIMD. It is written for the engineer who will run the test.

You can take one of these roles: the identity provider (IdP) that issues ID-JAGs, the authorization server (AS) that redeems them, or the MCP client, gateway or server that carries them. Gluu runs two public test servers that can play the IdP and the AS.

The demo story: a user (Bob) at a fictional advisory firm asks his AI agent to read a research service. The firm's IdP vouches for Bob with a short-lived ID-JAG, the research service's authorization server turns it into a 5-second access token, and the agent never holds Bob's tokens. Gluu has completed this flow end to end with Solo.io's agentgateway.

## Before you start

Tick these off on your side before the first call with Gluu.

- [ ] **A role.** Decide which role your product plays: IdP (issues ID-JAGs), authorization server (redeems ID-JAGs), MCP gateway or client, or MCP server
- [ ] **A public HTTPS issuer.** If your product is an IdP or AS, it needs an issuer URL reachable from the internet, with a discovery document. A server on a private or lab network cannot be reached by Gluu's servers
- [ ] **Signing keys.** A key pair for each client that will authenticate to Gluu (RSA or EC), kept private on your side, with the public key as a JWKS file
- [ ] **If you issue ID-JAGs:** the signing key's public certificate (PEM) and its key ID (`kid`)
- [ ] **Time sync.** Your servers use NTP. Tokens in the test live only a few seconds, so clocks must agree to about a second
- [ ] **A test user.** An account at your side that maps to one of Gluu's demo users, by email or subject
- [ ] **An engineer** who can join short calls and read decoded tokens (a JWT decoder is enough)
- [ ] **A log plan.** The ability to export request and response logs with tokens redacted, as evidence

## What to send us

Send only public information. We never need a private key, a password or a client secret from you.

| Item | Format | Why we need it |
| --- | --- | --- |
| Your role and the Gluu server you want to pair with | A short note | We configure the right server and test list |
| Technical contact and time zone | Name and email | To schedule short calls and share results |
| Issuer URL | `https://...` exactly as in your discovery document | Gluu trusts your issuer by name, so it must match character for character |
| Discovery document URL | `.../.well-known/openid-configuration` or `.../oauth-authorization-server` | We confirm your endpoints and your `jwks_uri` |
| Client ID | The exact `client_id` your product will present | We register a client with this ID. It must equal the `client_id` your ID-JAG carries |
| Client public keys | A JWKS file (public keys only), with `kid` and algorithm for each key | We register your client with `private_key_jwt`, so no shared secret is needed |
| Signing certificate (if you issue ID-JAGs) | PEM file plus the `kid` of the key you sign with | Gluu verifies the ID-JAG signature with keys it holds, so we import this certificate |
| Audience | The issuer of the authorization server that will redeem the ID-JAG | Becomes the `aud` claim |
| Scopes | The scopes you will request | We create and allow them on the client |
| User mapping | How your ID-JAG names the user (`sub`, `email` or `sub_id`) and the matching demo user | Gluu resolves the user in the order `sub`, then `email`, then `aud_sub` |
| Redirect URIs | Exact sign-in and sign-out URIs, if a browser sign-in is part of the test | We register them on the client |
| Token lifetimes | The ID-JAG and access-token lifetimes you expect | We align ours with yours |

## What we send you

After the intake call and once we have your public keys, Gluu sends the following. Anything secret goes by a separate channel from the rest.

| Item | Example or detail | How it arrives |
| --- | --- | --- |
| Issuer URL of each Gluu server you test with | `https://sterling-idp.gluu.info` (IdP) and `https://ledger-demo.gluu.info` (authorization server) | In the setup email |
| Endpoints | Token, discovery and JWKS URLs (see the reference section) | In the setup email |
| Your registered client ID at Gluu | The ID you sent us, confirmed back | In the setup email |
| Scopes registered for your client | For example `research:read` | In the setup email |
| Trusted-issuer confirmation | Your issuer added to the list of issuers the Gluu authorization server accepts | In the setup email |
| Demo user | A user such as `bob@sterling.lab` with a verified email, and how to sign in (passkey by default, password fallback) | Account name in the email; any password by a separate channel |
| Gluu's public signing key IDs | The `kid` Gluu signs ID-JAGs with, and its JWKS URL | In the setup email |
| Directory access (if SCIM is part of your test) | A client ID with read scopes for users and groups | Client ID in the email; secret by a separate channel |
| Test plan | The test matrix from the testing section, pre-filled with your details | Shared document |
| Results | The completed matrix and our logs, with tokens redacted | Shared document, after the test |

## How Gluu is configured for your test

Gluu sets up its side for you, based on the role you play. In every case your client authenticates with a signed JWT (`private_key_jwt`) and no shared secret.

| If your product is... | What Gluu configures | What you configure |
| --- | --- | --- |
| **An IdP that issues ID-JAGs** (paired with `ledger-demo`) | A client with the `client_id` you give us, your public keys, and the JWT bearer grant only. A scope for your tests. Your issuer added to the trusted-issuer list. Your signing certificate imported under its `kid`. A demo user matched to your user by email or subject. Access tokens issued as short-lived JWTs | Issue ID-JAGs with header `typ` `oauth-id-jag+jwt`, with `iss` (your issuer), `sub`, `aud` (the Gluu server's issuer), `client_id` (the ID we registered), `jti`, `iat` and a short `exp`. Sign the client assertion with your key; its `aud` is the Gluu issuer |
| **An authorization server that redeems ID-JAGs** (paired with `sterling-idp`) | A client with the `client_id` your AS expects, your public keys, and the token-exchange grant. Your audience and scope allowed. An Identity Assertion script for your audience, to set the `client_id` and any extra claims (for example a tenant) in the ID-JAG | Trust Gluu's issuer and fetch its keys from the JWKS URL. Verify `typ`, signature, `aud`, `client_id`, `exp` and `jti` |
| **An MCP gateway or MCP client** | Sign-in at `sterling-idp` with authorization code and PKCE S256, `private_key_jwt`, and the claims you need in the ID token (email, verified email, groups). A token endpoint for the exchange and a redemption endpoint at `ledger-demo` | Perform the sign-in, request the ID-JAG, redeem it, and call the tool before the token expires |
| **An MCP server** | Gluu issues the access token: a JWT with an agreed `aud`, scope and lifetime | Return 401 with a pointer to your protected-resource metadata, validate Gluu's JWT against its JWKS, and check `aud` and scope |
| **A directory or HR system** | Users and groups at `sterling-idp`, a SCIM v2 endpoint and a client with read scopes | Fetch a token with `client_credentials` and read users and groups. The user's verified-email flag is in the Gluu SCIM extension |

## Gluu endpoints and settings reference

Both servers run Janssen. Always read the live discovery document, because it is the source of truth for the values below.

| | `sterling-idp` (IdP) | `ledger-demo` (authorization server) |
| --- | --- | --- |
| Issuer | `https://sterling-idp.gluu.info` | `https://ledger-demo.gluu.info` |
| Discovery | `https://sterling-idp.gluu.info/.well-known/openid-configuration` | `https://ledger-demo.gluu.info/.well-known/openid-configuration` |
| Authorization-server metadata alias | `https://sterling-idp.gluu.info/.well-known/oauth-authorization-server` | Not available yet |
| Authorization endpoint | `https://sterling-idp.gluu.info/jans-auth/restv1/authorize` | `https://ledger-demo.gluu.info/jans-auth/restv1/authorize` |
| Token endpoint | `https://sterling-idp.gluu.info/jans-auth/restv1/token` | `https://ledger-demo.gluu.info/jans-auth/restv1/token` |
| Keys (JWKS) | `https://sterling-idp.gluu.info/jans-auth/restv1/jwks` | `https://ledger-demo.gluu.info/jans-auth/restv1/jwks` |
| SCIM v2 | `https://sterling-idp.gluu.info/jans-scim/restv1/v2` | Not part of the test setup |
| Client authentication | `private_key_jwt` (RS256, PS256 and other algorithms listed in discovery) | `private_key_jwt`, `tls_client_auth` and others listed in discovery |
| Grants of interest | Authorization code with PKCE S256, token exchange, JWT bearer, client credentials | JWT bearer, authorization code, client credentials, token exchange |
| PKCE | `S256` advertised in discovery | Supported, but not yet advertised in discovery |
| Software | Janssen 2.5.0 | Janssen 2.4.1 |

### ID-JAG issued by the Gluu IdP

A token exchange at the IdP's token endpoint (`grant_type` `urn:ietf:params:oauth:grant-type:token-exchange`, `requested_token_type` `urn:ietf:params:oauth:token-type:id-jag`, an ID token as `subject_token`, `audience` set to the target issuer) returns an ID-JAG with these properties.

| Claim or header | Value |
| --- | --- |
| `typ` (header) | `oauth-id-jag+jwt` |
| `alg` | RS256 |
| `iss` | `https://sterling-idp.gluu.info` |
| `aud` | The audience you request (the target AS issuer) |
| `client_id` | The client ID of the requesting client, or the value set for your audience by the Identity Assertion script |
| `sub`, `jti`, `iat`, `exp` | Present. Default lifetime 300 seconds, adjustable for the test |

### Access token issued by the Gluu authorization server

| Claim | Value |
| --- | --- |
| Format | JWT, RS256 |
| `iss` | `https://ledger-demo.gluu.info` |
| `aud` | Includes the agreed resource audience |
| `scope` | The granted scope, configured as a space-separated string (to be confirmed in your test) |
| `client_id` | The client ID that redeemed the ID-JAG |
| Lifetime | 5 seconds in the demo, with no refresh token |

## Step-by-step testing

The test runs in seven steps. Gluu is available on a call for steps 3 to 5.

1. **Intake (both).** Agree your role, the pairing and the test list. You send the items from "What to send us".
2. **Pre-flight (you).** Check that both sides can reach each other's metadata: `curl -s <issuer>/.well-known/openid-configuration` returns 200 with the exact issuer, and the `jwks_uri` returns keys.
3. **Gluu setup (Gluu).** Gluu registers your client, trust and demo user, then sends you the setup email.
4. **Smoke test (you).** Run the shortest path first: redeem one ID-JAG at the Gluu authorization server, or request one from the Gluu IdP. Expect one correct token or one readable error.
5. **Full flow (both).** Run the end-to-end scenario with your product and the demo user, within the token lifetime.
6. **Evidence (both).** Each side fills in the matrix below for the other, using only what it observed, with redacted logs.
7. **Sign-off (both).** The two matrices must agree. Gluu then removes your test client and imported certificates, and you remove any Gluu test data.

### Test cases

| Test | Pass criteria |
| --- | --- |
| Discovery | HTTP 200, `issuer` equals the URL exactly |
| Client authentication | Token request with `private_key_jwt` succeeds with no shared secret |
| ID-JAG issuance | `typ` is `oauth-id-jag+jwt`; `aud`, `client_id`, `sub`, `jti`, `iat` and a short `exp` are present |
| ID-JAG redemption | JWT bearer grant returns an access token for the right user |
| Token shape | Expected `aud`, scope and lifetime, and no refresh token |
| Wrong audience | An ID-JAG for another issuer is rejected |
| Wrong client | Redeeming with a different `client_id` is rejected |
| Replay | Reusing an ID-JAG is rejected (we will confirm this behavior with you in the first run) |
| CIMD | An HTTPS metadata URL as `client_id` with PKCE S256 returns an authorization code and redirects only to a URI listed in the metadata |
| MCP flow | The MCP server returns 401 with a metadata pointer, then accepts a valid Gluu token |

In the matrix, mark a row only if the logs show it. If one side cannot test a feature, mark it "unable to test"; a failed or untested feature stays blank.

## Troubleshooting

Common problems from earlier tests, with what to check first. Send us the full error response with tokens redacted if none of these fit.

| Symptom | Likely cause | What to check |
| --- | --- | --- |
| "ID-JAG signature verification failed" | Gluu does not hold your signing key, or a key rotation dropped it | Confirm the `kid` in your ID-JAG header matches the certificate Gluu imported. Ask Gluu to re-import it |
| ID-JAG rejected: issuer not trusted | Your issuer is not in Gluu's trusted-issuer list, or it differs by a character | Compare your `iss` with the issuer we registered, character for character, including any trailing slash |
| `aud` does not match | The ID-JAG audience is not the Gluu issuer | Use the exact issuer from the Gluu discovery document |
| `client_id` does not match the authenticated client | The ID-JAG `client_id` claim and the client that redeems it differ | Both must be the client ID registered at Gluu |
| Client authentication fails | Wrong `aud` in the client assertion, wrong key, or an algorithm Gluu does not expect | The assertion `aud` must equal the issuer. Check that the `kid` and algorithm match your registered JWKS |
| Token already expired | Clock difference or a slow call | Sync clocks with NTP and run the redemption within the token lifetime |
| User not found | No Gluu user matches your `sub` or `email` | Confirm the demo user's email, or send us the `sub` you use |
| Refused because of an unknown scope | The scope is not registered for your client | Ask Gluu to add it |
| Your AS cannot verify a Gluu ID-JAG | Your server cannot fetch Gluu's keys | Check that your server reaches the Gluu JWKS URL and trusts the Gluu issuer |

## Security and data handling

The Gluu servers are public test systems that use fictional demo users, so keep real personal data and production credentials out of the test.

1. **Public keys only.** Send JWKS files and certificates. Never email a private key, password or client secret.
2. **Separate channels.** If a flow needs a shared secret (for example SCIM), Gluu sends it in a different message from the setup details.
3. **Least privilege.** Your client gets only the grants and scopes the test needs, and test users have no administrative roles.
4. **Named trust.** Gluu adds your issuer by name to its trusted list. It does not accept issuers it was not told about.
5. **Short-lived tokens.** ID-JAGs and access tokens live seconds, and no refresh tokens are issued, unless a test needs more.
6. **Redact logs.** Replace tokens with a marker and keep the claims before sharing logs.
7. **No real data.** Use the fictional demo users. Do not load real user records into the test servers.
8. **Clean-up.** After sign-off, Gluu removes your client and imported certificates and rotates anything shared. Please remove Gluu's keys and test data from your systems.

## Contact and next steps

**Gluu contact:** Mohib Zico, Director of Enterprise Solution and Support, Gluu Inc. (mohib@gluu.org).

This testing supports the OpenID Foundation AIIM Community Group's MCP security interoperability event, whose results are presented at the Gartner IAM Summit on December 7-9, 2026. The event runs weekly coordination calls (Mondays, 11 AM Pacific, until November 30), and your results with Gluu can count toward it.

**To get started**

- [ ] Reply with your role and the Gluu server you want to pair with
- [ ] Send your technical contact and time zone
- [ ] Send the items listed in "What to send us" (public information only)
- [ ] Book the intake call
- [ ] Receive the setup email from Gluu and run the pre-flight checks
- [ ] Run the smoke test and the full flow
- [ ] Agree the test matrix and the logs
- [ ] Confirm clean-up when testing ends
