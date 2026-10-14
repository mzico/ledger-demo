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
| OAuth AS | Descope: Agentic Identity Hub (Cross-App Access) | Kevin Gao | kevin@descope.com |
| OpenID Provider | Gluu: Gluu Flex / Janssen (sterling-idp.gluu.info) | Mohibuzzaman Zico | zico@gluu.org |

## Test Report
The OpenID Provider section is removed (Gluu's own role).

### MCP Client Operation
| Title | MCP Client (Solo.io) |
|---|---|
| Valid ID-JAG | ✅ (Descope validated the ID-JAG that the client presented: log 4) |
| resource support | ✅ (log 1) |
| scope support | ✅ (log 1) |

### OAuth AS Operation
| Title | OAuth AS (Descope) |
|---|---|
| Valid ID-JAG | ✅ (four IDJAGValidated events in Descope's audit log: log 4; access token issued: log 3) |
| resource support | ✅ (`resource` in the IDJAGValidated event and in the access token audience: logs 3 and 4) |
| sub_id support | (blank: Gluu does not issue sub_id) |
| scope support | ✅ (`scopes: research:read` in the IDJAGValidated event; response scope=research:read: logs 3 and 4) |

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
client_id: <descope-client-id>
email: bob@sterling.lab
exp: 1791909651
iat: 1791909351
iss: https://sterling-idp.gluu.info
jti: 1129ceec-4112-4e5a-bde6-e5165caf151c
resource: https://mcp.ledgerline.lab/mcp
scope: research:read
sub: 9HX4wEtMFfR7ViNrDMHp1DwQto6qdkdnJhG6ZDn1Hjo
Requested in the token exchange: requested_token_type=urn:ietf:params:oauth:token-type:id-jag, audience=https://api.descope.com/v1/apps/agentic/P3KZmqLCLPjIQSkiFVdVP2c9SXnG/MS3KZnUJNb8zY67xgs36m9Gg9ekrz, resource=https://mcp.ledgerline.lab/mcp, scope=research:read
Checks: typ oauth-id-jag+jwt; aud "https://api.descope.com/v1/apps/agentic/P3KZmqLCLPjIQSkiFVdVP2c9SXnG/MS3KZnUJNb8zY67xgs36m9Gg9ekrz"; client_id <descope-client-id>; resource https://mcp.ledgerline.lab/mcp; scope research:read; sub_id not issued (sub only); sid not issued
Descope accepted the ID-JAG: POST /xaa-legs/ledgerline/as/token (grant_type=urn:ietf:params:oauth:grant-type:jwt-bearer) -> 200 at 2026-10-13T16:35:51.100Z (121ms)
response: token_type=Bearer, issued_token_type=urn:ietf:params:oauth:token-type:access_token, scope=research:read, expires_in=600
access token header: {"alg":"RS256","kid":"SK3KZmqInVTjBgwwl0IFKRuDF2mAv","typ":"JWT"}
access token payload: {"iss":"https://api.descope.com/v1/apps/agentic/P3KZmqLCLPjIQSkiFVdVP2c9SXnG/MS3KZnUJNb8zY67xgs36m9Gg9ekrz","sub":"U3KbGNm0kEcNUYsXuSxXh6rD8bQG","aud":["<descope-client-id>","P3KZmqLCLPjIQSkiFVdVP2c9SXnG","https://mcp.ledgerline.lab/mcp"],"scope":"research:read","exp":1791909951,"iat":1791909351}
```

**Log 4. Descope (OAuth AS), audit log: four IDJAGValidated events, each with the validated ID-JAG's claims (provided by Solo.io and Descope)**
```
IDJAGValidated 2026-10-13T16:35:40Z data {"audience":"https://api.descope.com/v1/apps/agentic/P3KZmqLCLPjIQSkiFVdVP2c9SXnG/MS3KZnUJNb8zY67xgs36m9Gg9ekrz","client_id":"<descope-client-id>","correlation_id":"3KeDrgDAw41sZ3rY13m3p5cFQLf","device":"Bot","external_issuer":"https://sterling-idp.gluu.info","external_subject":"9HX4wEtMFfR7ViNrDMHp1DwQto6qdkdnJhG6ZDn1Hjo-xaatestsv","request_details":{"contentLength":"1303","host":"api.descope.com","method":"POST","uri":"/oauth2/v1/apps/agentic/P3KZmqLCLPjIQSkiFVdVP2c9SXnG/MS3KZnUJNb8zY67xgs36m9Gg9ekrz/token"},"resource":"https://mcp.ledgerline.lab/mcp","scopes":["research:read"],"third_party_application_id":"<descope-client-id>"}
validated ID-JAG {"acr":"fido2","amr":["70"],"aud":["https://api.descope.com/v1/apps/agentic/P3KZmqLCLPjIQSkiFVdVP2c9SXnG/MS3KZnUJNb8zY67xgs36m9Gg9ekrz"],"aud_tenant":"xaatestsv","auth_time":1791907971,"client_id":"<descope-client-id>","email":"bob@sterling.lab","exp":"2026-10-13T16:40:40Z","iat":"2026-10-13T16:35:40Z","iss":"https://sterling-idp.gluu.info","jti":"fb07d9ae-a860-480b-9357-520747918bb6","resource":"https://mcp.ledgerline.lab/mcp","scope":"research:read","sub":"9HX4wEtMFfR7ViNrDMHp1DwQto6qdkdnJhG6ZDn1Hjo"}
IDJAGValidated 2026-10-13T16:35:41Z data {"audience":"https://api.descope.com/v1/apps/agentic/P3KZmqLCLPjIQSkiFVdVP2c9SXnG/MS3KZnUJNb8zY67xgs36m9Gg9ekrz","client_id":"<descope-client-id>","correlation_id":"3KeDrnkzylXOlfIeq7XTRv4hLVW","device":"Bot","external_issuer":"https://sterling-idp.gluu.info","external_subject":"9HX4wEtMFfR7ViNrDMHp1DwQto6qdkdnJhG6ZDn1Hjo-xaatestsv","request_details":{"contentLength":"1303","host":"api.descope.com","method":"POST","uri":"/oauth2/v1/apps/agentic/P3KZmqLCLPjIQSkiFVdVP2c9SXnG/MS3KZnUJNb8zY67xgs36m9Gg9ekrz/token"},"resource":"https://mcp.ledgerline.lab/mcp","scopes":["research:read"],"third_party_application_id":"<descope-client-id>"}
validated ID-JAG {"acr":"fido2","amr":["70"],"aud":["https://api.descope.com/v1/apps/agentic/P3KZmqLCLPjIQSkiFVdVP2c9SXnG/MS3KZnUJNb8zY67xgs36m9Gg9ekrz"],"aud_tenant":"xaatestsv","auth_time":1791907971,"client_id":"<descope-client-id>","email":"bob@sterling.lab","exp":"2026-10-13T16:40:41Z","iat":"2026-10-13T16:35:41Z","iss":"https://sterling-idp.gluu.info","jti":"b206f17b-1dc9-45aa-adc5-a4a0c510f04d","resource":"https://mcp.ledgerline.lab/mcp","scope":"research:read","sub":"9HX4wEtMFfR7ViNrDMHp1DwQto6qdkdnJhG6ZDn1Hjo"}
IDJAGValidated 2026-10-13T16:35:41Z data {"audience":"https://api.descope.com/v1/apps/agentic/P3KZmqLCLPjIQSkiFVdVP2c9SXnG/MS3KZnUJNb8zY67xgs36m9Gg9ekrz","client_id":"<descope-client-id>","correlation_id":"3KeDrpxOIgmzSPrX0gH27IclWaK","device":"Bot","external_issuer":"https://sterling-idp.gluu.info","external_subject":"9HX4wEtMFfR7ViNrDMHp1DwQto6qdkdnJhG6ZDn1Hjo-xaatestsv","request_details":{"contentLength":"1303","host":"api.descope.com","method":"POST","uri":"/oauth2/v1/apps/agentic/P3KZmqLCLPjIQSkiFVdVP2c9SXnG/MS3KZnUJNb8zY67xgs36m9Gg9ekrz/token"},"resource":"https://mcp.ledgerline.lab/mcp","scopes":["research:read"],"third_party_application_id":"<descope-client-id>"}
validated ID-JAG {"acr":"fido2","amr":["70"],"aud":["https://api.descope.com/v1/apps/agentic/P3KZmqLCLPjIQSkiFVdVP2c9SXnG/MS3KZnUJNb8zY67xgs36m9Gg9ekrz"],"aud_tenant":"xaatestsv","auth_time":1791907971,"client_id":"<descope-client-id>","email":"bob@sterling.lab","exp":"2026-10-13T16:40:41Z","iat":"2026-10-13T16:35:41Z","iss":"https://sterling-idp.gluu.info","jti":"2614b614-db82-4a82-8a7a-bb8ccae13a72","resource":"https://mcp.ledgerline.lab/mcp","scope":"research:read","sub":"9HX4wEtMFfR7ViNrDMHp1DwQto6qdkdnJhG6ZDn1Hjo"}
IDJAGValidated 2026-10-13T16:35:51Z data {"audience":"https://api.descope.com/v1/apps/agentic/P3KZmqLCLPjIQSkiFVdVP2c9SXnG/MS3KZnUJNb8zY67xgs36m9Gg9ekrz","client_id":"<descope-client-id>","correlation_id":"3KeDt8tfJM2N5GpBveRL3br1H1M","device":"Bot","external_issuer":"https://sterling-idp.gluu.info","external_subject":"9HX4wEtMFfR7ViNrDMHp1DwQto6qdkdnJhG6ZDn1Hjo-xaatestsv","request_details":{"contentLength":"1303","host":"api.descope.com","method":"POST","uri":"/oauth2/v1/apps/agentic/P3KZmqLCLPjIQSkiFVdVP2c9SXnG/MS3KZnUJNb8zY67xgs36m9Gg9ekrz/token"},"resource":"https://mcp.ledgerline.lab/mcp","scopes":["research:read"],"third_party_application_id":"<descope-client-id>"}
validated ID-JAG {"acr":"fido2","amr":["70"],"aud":["https://api.descope.com/v1/apps/agentic/P3KZmqLCLPjIQSkiFVdVP2c9SXnG/MS3KZnUJNb8zY67xgs36m9Gg9ekrz"],"aud_tenant":"xaatestsv","auth_time":1791907971,"client_id":"<descope-client-id>","email":"bob@sterling.lab","exp":"2026-10-13T16:40:51Z","iat":"2026-10-13T16:35:51Z","iss":"https://sterling-idp.gluu.info","jti":"1129ceec-4112-4e5a-bde6-e5165caf151c","resource":"https://mcp.ledgerline.lab/mcp","scope":"research:read","sub":"9HX4wEtMFfR7ViNrDMHp1DwQto6qdkdnJhG6ZDn1Hjo"}
```

## Notes and disclosures
- sterling-idp.gluu.info runs Janssen 2.5.0. ledger-demo.gluu.info runs Janssen 2.4.1 (Flex).
- Token values (client_assertion, assertion, subject_token, code, code_verifier, state, code_challenge) are replaced by `<redacted>`. Everything else is raw jans-auth.log output.
- The Descope application `client_id` is shown as `<descope-client-id>` in this copy. The value is in the MCP client's report.
- This report covers the 16:35:40 to 16:35:51 UTC run only, matching the MCP client's report.
