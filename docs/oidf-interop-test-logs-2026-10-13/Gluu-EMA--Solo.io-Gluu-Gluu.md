# MCP Security Interoperability Test (SIT) Dec 2026
## EMA Interoperability Testing Log
File name: `Gluu-EMA--Solo.io-Gluu-Gluu`
Tester: Gluu (Mohibuzzaman Zico, zico@gluu.org). All times UTC. Both Gluu servers ran with DEBUG logging for the test.

## When Performed
Started: 2026-10-13 16:51:16 UTC
Ended: 2026-10-13 16:52:27 UTC
Where performed: online

## Participants
| Role | Company and Product | Representative | Email |
|---|---|---|---|
| MCP Client | Solo.io: Solo Enterprise for agentgateway, kagent Enterprise | Nick Gamb | nick.gamb@solo.io |
| OAuth AS | Gluu: Gluu Flex / Janssen (ledger-demo.gluu.info) | Mohibuzzaman Zico | zico@gluu.org |
| OpenID Provider | Gluu: Gluu Flex / Janssen (sterling-idp.gluu.info) | Mohibuzzaman Zico | zico@gluu.org |

## Test Report
### MCP Client Operation
Gluu holds both the OAuth AS and the OpenID Provider roles in this test, so the same person verified both sides. All marks are provisional (☑️). The OAuth AS and OpenID Provider sections are removed.

| Title | MCP Client (Solo.io) |
|---|---|
| Valid ID-JAG | ☑️ (ledger-demo validated the ID-JAG the client presented: logs 3 and 4) |
| resource support | ☑️ (the client sent `resource` in the token exchange: log 1; the issued ID-JAG carries it: log 5) |
| scope support | ☑️ (the client sent `scope=research:read` in both requests: logs 1 and 3; the ID-JAG carries it: log 5) |

## Test Logs, Screenshots and Videos
Representative exchange at 16:51:16 to 16:51:17 UTC. The same sequence repeats three more times at 16:51:17 and once at 16:52:26 UTC (shown last).

**Log 1. sterling-idp (OpenID Provider), 16:51:16. RFC 8693 token exchange (private_key_jwt client authentication)**
```
2026-10-13 16:51:16,912 DEBUG [qtp1783047508-16] 83d16253-6f4d-4ef2-96a0-6b2e21688f8e [as.server.token.ws.rs.TokenRestWebServiceImpl] (TokenRestWebServiceImpl.java:171) - Attempting to request access token: grantType = urn:ietf:params:oauth:grant-type:token-exchange, code = <redacted>, redirectUri = null, username = null, refreshToken = null, clientId = fcffc488-b086-4a69-a6c5-47d15c4f00c4, ExtraParams = {subject_token_type=[urn:ietf:params:oauth:token-type:id_token], audience=[https://ledger-demo.gluu.info], client_assertion_type=[urn:ietf:params:oauth:client-assertion-type:jwt-bearer], grant_type=[urn:ietf:params:oauth:grant-type:token-exchange], resource=[https://mcp.ledgerline.lab/mcp], scope=[research:read], client_assertion=<redacted>, subject_token=<redacted>, requested_token_type=[urn:ietf:params:oauth:token-type:id-jag], client_id=[fcffc488-b086-4a69-a6c5-47d15c4f00c4]}, isSecure = true, codeVerifier = <redacted>, ticket = null, authorizationDetails = null
```

**Log 2. sterling-idp, ID-JAG issued with audience ledger-demo**
```
2026-10-13 16:51:16,916 DEBUG [qtp1783047508-16] 83d16253-6f4d-4ef2-96a0-6b2e21688f8e [as.server.token.ws.rs.IdJagService] (IdJagService.java:118) - Issued ID-JAG for client: fcffc488-b086-4a69-a6c5-47d15c4f00c4, audience: https://ledger-demo.gluu.info
```

**Log 3. ledger-demo (OAuth AS), 16:51:17. jwt-bearer grant (private_key_jwt client authentication)**
```
2026-10-13 16:51:17,061 DEBUG [qtp1176735295-17] 9f2c97ba-a5b3-47a4-827e-d0437e53b609 [as.server.token.ws.rs.TokenRestWebServiceImpl] (TokenRestWebServiceImpl.java:170) - Attempting to request access token: grantType = urn:ietf:params:oauth:grant-type:jwt-bearer, code = <redacted>, redirectUri = null, username = null, refreshToken = null, clientId = fcffc488-b086-4a69-a6c5-47d15c4f00c4, ExtraParams = {client_assertion_type=[urn:ietf:params:oauth:client-assertion-type:jwt-bearer], grant_type=[urn:ietf:params:oauth:grant-type:jwt-bearer], scope=[research:read], assertion=<redacted>, client_assertion=<redacted>, client_id=[fcffc488-b086-4a69-a6c5-47d15c4f00c4]}, isSecure = true, codeVerifier = <redacted>, ticket = null, authorizationDetails = null
```

**Log 4. ledger-demo, ID-JAG validated, user resolved, scope granted**
```
2026-10-13 16:51:17,063 DEBUG [qtp1176735295-17] 9f2c97ba-a5b3-47a4-827e-d0437e53b609 [as.server.token.ws.rs.IdJagValidatorService] (IdJagValidatorService.java:88) - Validating ID-JAG for client: fcffc488-b086-4a69-a6c5-47d15c4f00c4
2026-10-13 16:51:17,068 DEBUG [qtp1176735295-17] 9f2c97ba-a5b3-47a4-827e-d0437e53b609 [as.server.token.ws.rs.IdJagValidatorService] (IdJagValidatorService.java:231) - Resolved user by 'email', user: bob
2026-10-13 16:51:17,068 DEBUG [qtp1176735295-17] 9f2c97ba-a5b3-47a4-827e-d0437e53b609 [as.server.token.ws.rs.IdJagValidatorService] (IdJagValidatorService.java:100) - ID-JAG validated successfully for client: fcffc488-b086-4a69-a6c5-47d15c4f00c4
2026-10-13 16:51:17,070 DEBUG [qtp1176735295-17] 9f2c97ba-a5b3-47a4-827e-d0437e53b609 [jans.as.server.model.authorize.ScopeChecker] (ScopeChecker.java:91) - Granted scopes: [research:read]
```

**Log 5. Decoded ID-JAG and ledger-demo's response, from the MCP client's gateway log (provided by Solo.io, signature removed)**
```
ID-JAG issued by Gluu (sterling-idp.gluu.info) at 2026-10-13T16:52:25.993Z for the account-info tool call (2026-10-13T16:52:27.034Z, trace 1efeb728a02b7b6c1c197ea4df2b4618)
header: {"kid":"connect_506b48d1-79c3-45b1-8463-eddda3a5c41d_sig_rs256","typ":"oauth-id-jag+jwt","alg":"RS256"}
payload:
acr: fido2
amr: ["70"]
aud: https://ledger-demo.gluu.info
auth_time: 1791910247
client_id: fcffc488-b086-4a69-a6c5-47d15c4f00c4
email: bob@sterling.lab
exp: 1791910646
iat: 1791910346
iss: https://sterling-idp.gluu.info
jti: 23eec5d8-0105-4a75-8fd3-1af7d2094de4
resource: https://mcp.ledgerline.lab/mcp
scope: research:read
sub: 9HX4wEtMFfR7ViNrDMHp1DwQto6qdkdnJhG6ZDn1Hjo
Requested in the token exchange: requested_token_type=urn:ietf:params:oauth:token-type:id-jag, audience=https://ledger-demo.gluu.info, resource=https://mcp.ledgerline.lab/mcp, scope=research:read
Checks: typ oauth-id-jag+jwt; aud "https://ledger-demo.gluu.info"; client_id fcffc488-b086-4a69-a6c5-47d15c4f00c4; resource https://mcp.ledgerline.lab/mcp; scope research:read; sub_id not issued (sub only); sid not issued
Gluu ledger-demo accepted the ID-JAG: POST /xaa-legs/ledgerline/as/token (grant_type=urn:ietf:params:oauth:grant-type:jwt-bearer) -> 200 at 2026-10-13T16:52:26.133Z (137ms)
response: token_type=Bearer, issued_token_type=urn:ietf:params:oauth:token-type:access_token, scope=research:read, expires_in=5
access token header: {"kid":"connect_261dceaa-e858-4872-b61a-6bc54aa209e8_sig_rs256","typ":"JWT","alg":"RS256"}
access token payload: {"iss":"https://ledger-demo.gluu.info","sub":"004fb040-80df-4abf-8f81-eee69263cf55","aud":["fcffc488-b086-4a69-a6c5-47d15c4f00c4","ledgerline-research"],"client_id":"fcffc488-b086-4a69-a6c5-47d15c4f00c4","scope":"[research:read]","exp":1791910351,"iat":1791910346}
```

**Second exchange, 16:52:26 UTC**

**sterling-idp**
```
2026-10-13 16:52:26,025 DEBUG [qtp1783047508-20] 3a386c38-cc6a-45c5-9bbb-f78e6299a094 [as.server.token.ws.rs.TokenRestWebServiceImpl] (TokenRestWebServiceImpl.java:171) - Attempting to request access token: grantType = urn:ietf:params:oauth:grant-type:token-exchange, code = <redacted>, redirectUri = null, username = null, refreshToken = null, clientId = fcffc488-b086-4a69-a6c5-47d15c4f00c4, ExtraParams = {subject_token_type=[urn:ietf:params:oauth:token-type:id_token], audience=[https://ledger-demo.gluu.info], client_assertion_type=[urn:ietf:params:oauth:client-assertion-type:jwt-bearer], grant_type=[urn:ietf:params:oauth:grant-type:token-exchange], resource=[https://mcp.ledgerline.lab/mcp], scope=[research:read], client_assertion=<redacted>, subject_token=<redacted>, requested_token_type=[urn:ietf:params:oauth:token-type:id-jag], client_id=[fcffc488-b086-4a69-a6c5-47d15c4f00c4]}, isSecure = true, codeVerifier = <redacted>, ticket = null, authorizationDetails = null
2026-10-13 16:52:26,028 DEBUG [qtp1783047508-20] 3a386c38-cc6a-45c5-9bbb-f78e6299a094 [as.server.token.ws.rs.IdJagService] (IdJagService.java:118) - Issued ID-JAG for client: fcffc488-b086-4a69-a6c5-47d15c4f00c4, audience: https://ledger-demo.gluu.info
```

**ledger-demo**
```
2026-10-13 16:52:26,161 DEBUG [qtp1176735295-20] 7eb0ff4b-b462-42c7-ba22-52d636eb2b3e [as.server.token.ws.rs.TokenRestWebServiceImpl] (TokenRestWebServiceImpl.java:170) - Attempting to request access token: grantType = urn:ietf:params:oauth:grant-type:jwt-bearer, code = <redacted>, redirectUri = null, username = null, refreshToken = null, clientId = fcffc488-b086-4a69-a6c5-47d15c4f00c4, ExtraParams = {client_assertion_type=[urn:ietf:params:oauth:client-assertion-type:jwt-bearer], grant_type=[urn:ietf:params:oauth:grant-type:jwt-bearer], scope=[research:read], assertion=<redacted>, client_assertion=<redacted>, client_id=[fcffc488-b086-4a69-a6c5-47d15c4f00c4]}, isSecure = true, codeVerifier = <redacted>, ticket = null, authorizationDetails = null
2026-10-13 16:52:26,162 DEBUG [qtp1176735295-20] 7eb0ff4b-b462-42c7-ba22-52d636eb2b3e [as.server.token.ws.rs.IdJagValidatorService] (IdJagValidatorService.java:88) - Validating ID-JAG for client: fcffc488-b086-4a69-a6c5-47d15c4f00c4
2026-10-13 16:52:26,166 DEBUG [qtp1176735295-20] 7eb0ff4b-b462-42c7-ba22-52d636eb2b3e [as.server.token.ws.rs.IdJagValidatorService] (IdJagValidatorService.java:231) - Resolved user by 'email', user: bob
2026-10-13 16:52:26,166 DEBUG [qtp1176735295-20] 7eb0ff4b-b462-42c7-ba22-52d636eb2b3e [as.server.token.ws.rs.IdJagValidatorService] (IdJagValidatorService.java:100) - ID-JAG validated successfully for client: fcffc488-b086-4a69-a6c5-47d15c4f00c4
2026-10-13 16:52:26,168 DEBUG [qtp1176735295-20] 7eb0ff4b-b462-42c7-ba22-52d636eb2b3e [jans.as.server.model.authorize.ScopeChecker] (ScopeChecker.java:91) - Granted scopes: [research:read]
```

## Notes and disclosures
- sterling-idp.gluu.info runs Janssen 2.5.0. ledger-demo.gluu.info runs Janssen 2.4.1 (Flex).
- Token values (client_assertion, assertion, subject_token, code, code_verifier, state, code_challenge) are replaced by `<redacted>`. Everything else is raw jans-auth.log output.
- Janssen verifies an ID-JAG signature only with the server's own keystore. For this run the signing certificate of sterling-idp was imported into the keystore of ledger-demo (under its kid) as a workaround. Fetching the trusted issuer's keys from a jwks_uri or a static JWKS is not available yet.
- `sub_id` is not issued by Gluu (the ID-JAG carries `sub` only), so that cell is blank.
- Not verified: the MCP client does not send `resource` in the jwt-bearer request to the AS, so resource support at the AS is not exercised.
