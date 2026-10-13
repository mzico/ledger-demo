# MCP Security Interoperability Test (SIT) Dec 2026
## EMA Interoperability Testing Log
File name: `Gluu-EMA--Solo.io-Descope-Gluu`
Tester: Gluu (Mohibuzzaman Zico, zico@gluu.org). All times UTC. Both Gluu servers ran with DEBUG logging for the test.

## When Performed
Started: 2026-10-13 16:35:40 UTC
Ended: 2026-10-13 16:35:51 UTC
Where performed: online

## Participants
| Role | Company and Product | Representative | Email |
|---|---|---|---|
| MCP Client | Solo.io: Solo Enterprise for agentgateway, kagent Enterprise | Nick Gamb | nick.gamb@solo.io |
| OAuth AS | Descope: Agentic Identity Hub (Cross-App Access) | Nick Gamb (operates the Descope tenant) | nick.gamb@solo.io |
| OpenID Provider | Gluu: Gluu Flex / Janssen (sterling-idp.gluu.info) | Mohibuzzaman Zico | zico@gluu.org |

## Test Report
The OpenID Provider section is removed (Gluu's own role).

### MCP Client Operation
| Title | MCP Client (Solo.io) |
|---|---|
| Valid ID-JAG | (blank: Gluu issued the ID-JAG but did not see the client present it to Descope) |
| resource support | ✅ (log 1) |
| scope support | ✅ (log 1) |

### OAuth AS Operation
The MCP Client and the OAuth AS are operated by the same person (Nick Gamb), so the Descope marks are provisional (☑️). The evidence is the MCP client's gateway log (log 3), since Gluu cannot see Descope's side.

| Title | OAuth AS (Descope) |
|---|---|
| Valid ID-JAG | ☑️ (Descope accepted the Gluu ID-JAG: 200 with an access token, log 3) |
| resource support | ☑️ (the Descope access token audience includes the resource, log 3) |
| sub_id support | (blank: Gluu does not issue sub_id) |
| scope support | ☑️ (response scope=research:read, log 3) |

## Test Logs, Screenshots and Videos
**Log 1. sterling-idp (OpenID Provider), 16:35:40. RFC 8693 token exchange, audience is Descope**
```
2026-10-13 16:35:40,586 DEBUG [qtp1783047508-16] e7e8b6da-44c9-47fe-99de-407516ea0afa [as.server.token.ws.rs.TokenRestWebServiceImpl] (TokenRestWebServiceImpl.java:171) - Attempting to request access token: grantType = urn:ietf:params:oauth:grant-type:token-exchange, code = <redacted>, redirectUri = null, username = null, refreshToken = null, clientId = fcffc488-b086-4a69-a6c5-47d15c4f00c4, ExtraParams = {subject_token_type=[urn:ietf:params:oauth:token-type:id_token], audience=[https://api.descope.com/v1/apps/agentic/P3KZmqLCLPjIQSkiFVdVP2c9SXnG/MS3KZnUJNb8zY67xgs36m9Gg9ekrz], client_assertion_type=[urn:ietf:params:oauth:client-assertion-type:jwt-bearer], grant_type=[urn:ietf:params:oauth:grant-type:token-exchange], resource=[https://mcp.ledgerline.lab/mcp], scope=[research:read], client_assertion=<redacted>, subject_token=<redacted>, requested_token_type=[urn:ietf:params:oauth:token-type:id-jag], client_id=[fcffc488-b086-4a69-a6c5-47d15c4f00c4]}, isSecure = true, codeVerifier = <redacted>, ticket = null, authorizationDetails = null
```

**Log 2. sterling-idp, ID-JAG issued (16:35:40, 16:35:41, 16:35:41 and 16:35:51 UTC)**
```
2026-10-13 16:35:40,591 DEBUG [qtp1783047508-16] e7e8b6da-44c9-47fe-99de-407516ea0afa [as.server.token.ws.rs.IdJagService] (IdJagService.java:118) - Issued ID-JAG for client: fcffc488-b086-4a69-a6c5-47d15c4f00c4, audience: https://api.descope.com/v1/apps/agentic/P3KZmqLCLPjIQSkiFVdVP2c9SXnG/MS3KZnUJNb8zY67xgs36m9Gg9ekrz
2026-10-13 16:35:41,088 DEBUG [qtp1783047508-20] eb852427-f88a-41cc-8ed6-6b26aac73f53 [as.server.token.ws.rs.IdJagService] (IdJagService.java:118) - Issued ID-JAG for client: fcffc488-b086-4a69-a6c5-47d15c4f00c4, audience: https://api.descope.com/v1/apps/agentic/P3KZmqLCLPjIQSkiFVdVP2c9SXnG/MS3KZnUJNb8zY67xgs36m9Gg9ekrz
2026-10-13 16:35:41,476 DEBUG [qtp1783047508-16] 04c260cd-5d4f-40ca-b5e5-f337f7c4ebdf [as.server.token.ws.rs.IdJagService] (IdJagService.java:118) - Issued ID-JAG for client: fcffc488-b086-4a69-a6c5-47d15c4f00c4, audience: https://api.descope.com/v1/apps/agentic/P3KZmqLCLPjIQSkiFVdVP2c9SXnG/MS3KZnUJNb8zY67xgs36m9Gg9ekrz
2026-10-13 16:35:51,002 DEBUG [qtp1783047508-20] 0bf34d69-b11c-4d98-ba6b-62125baf928e [as.server.token.ws.rs.IdJagService] (IdJagService.java:118) - Issued ID-JAG for client: fcffc488-b086-4a69-a6c5-47d15c4f00c4, audience: https://api.descope.com/v1/apps/agentic/P3KZmqLCLPjIQSkiFVdVP2c9SXnG/MS3KZnUJNb8zY67xgs36m9Gg9ekrz
```

**Log 3. Decoded ID-JAG and Descope's response, from the MCP client's gateway log (provided by Solo.io, signature removed)**
```
ID-JAG issued by Gluu (sterling-idp.gluu.info) at 2026-10-13T16:35:50.975Z for the account-info tool call (2026-10-13T16:35:51.221Z, trace 01fd1b8067cb2403245e5f8c98704b47)
header: {"kid":"connect_506b48d1-79c3-45b1-8463-eddda3a5c41d_sig_rs256","typ":"oauth-id-jag+jwt","alg":"RS256"}
payload:
acr: fido2
amr: ["70"]
aud: https://api.descope.com/v1/apps/agentic/P3KZmqLCLPjIQSkiFVdVP2c9SXnG/MS3KZnUJNb8zY67xgs36m9Gg9ekrz
aud_tenant: xaatestsv
auth_time: 1791907971
client_id: <descope-client-id: pending confirmation>
email: bob@sterling.lab
exp: 1791909651
iat: 1791909351
iss: https://sterling-idp.gluu.info
jti: 1129ceec-4112-4e5a-bde6-e5165caf151c
resource: https://mcp.ledgerline.lab/mcp
scope: research:read
sub: 9HX4wEtMFfR7ViNrDMHp1DwQto6qdkdnJhG6ZDn1Hjo
Requested in the token exchange: requested_token_type=urn:ietf:params:oauth:token-type:id-jag, audience=https://api.descope.com/v1/apps/agentic/P3KZmqLCLPjIQSkiFVdVP2c9SXnG/MS3KZnUJNb8zY67xgs36m9Gg9ekrz, resource=https://mcp.ledgerline.lab/mcp, scope=research:read
Checks: typ oauth-id-jag+jwt; aud "https://api.descope.com/v1/apps/agentic/P3KZmqLCLPjIQSkiFVdVP2c9SXnG/MS3KZnUJNb8zY67xgs36m9Gg9ekrz"; client_id <descope-client-id: pending confirmation>; resource https://mcp.ledgerline.lab/mcp; scope research:read; sub_id not issued (sub only); sid not issued
Descope accepted the ID-JAG: POST /xaa-legs/ledgerline/as/token (grant_type=urn:ietf:params:oauth:grant-type:jwt-bearer) -> 200 at 2026-10-13T16:35:51.100Z (121ms)
response: token_type=Bearer, issued_token_type=urn:ietf:params:oauth:token-type:access_token, scope=research:read, expires_in=600
access token header: {"alg":"RS256","kid":"SK3KZmqInVTjBgwwl0IFKRuDF2mAv","typ":"JWT"}
access token payload: {"iss":"https://api.descope.com/v1/apps/agentic/P3KZmqLCLPjIQSkiFVdVP2c9SXnG/MS3KZnUJNb8zY67xgs36m9Gg9ekrz","sub":"U3KbGNm0kEcNUYsXuSxXh6rD8bQG","aud":["<descope-client-id: pending confirmation>","P3KZmqLCLPjIQSkiFVdVP2c9SXnG","https://mcp.ledgerline.lab/mcp"],"scope":"research:read","exp":1791909951,"iat":1791909351}
```

## Notes and disclosures
- sterling-idp.gluu.info runs Janssen 2.5.0. ledger-demo.gluu.info runs Janssen 2.4.1 (Flex).
- Token values (client_assertion, assertion, subject_token, code, code_verifier, state, code_challenge) are replaced by `<redacted>`. Everything else is raw jans-auth.log output.
- The Descope `client_id` value is left out of this copy until Solo.io confirms it is an identifier and not a secret.
- This report covers the 16:35:40 to 16:35:51 UTC run only, matching the MCP client's report.
