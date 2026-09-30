# CIMD & ID-JAG Test Guide

Two short tests against the Janssen server: (1) log in with a CIMD (URL) `client_id`, (2) mint an ID-JAG assertion. Each is a few copy-paste steps.

## Server

| Item | Value |
| --- | --- |
| Base URL | `https://ledger-demo.gluu.info` |
| Issuer | `https://ledger-demo.gluu.info` |
| Authorize endpoint | `https://ledger-demo.gluu.info/jans-auth/restv1/authorize` |
| Token endpoint | `https://ledger-demo.gluu.info/jans-auth/restv1/token` |
| Discovery | `https://ledger-demo.gluu.info/.well-known/openid-configuration` |
| Version | Janssen 2.4.1 |
| Features enabled | CIMD (`client_id_metadata_document`), ID-JAG (`identity_assertion_authz_grant`) |

## Before you start

You need three things:

1. The **server base URL** above (`https://ledger-demo.gluu.info`).
2. A browser and [oidcdebugger.com/debug](https://oidcdebugger.com/debug) as the redirect target.
3. A **login account** on the server (username + password we provided).

The CIMD test also needs a public HTTPS URL that returns your client metadata (a GitHub gist works).

## Test 1 — CIMD login

Goal: log in using a URL as the `client_id`. No client pre-registration.

1. **Host your metadata document** at a public HTTPS URL. Minimum content:

    ```json
    {
      "client_id": "https://YOUR-METADATA-URL",
      "redirect_uris": ["https://oidcdebugger.com/debug"],
      "grant_types": ["authorization_code"],
      "response_types": ["code"],
      "scope": "openid",
      "token_endpoint_auth_method": "none"
    }
    ```

    The `client_id` value must equal the URL you host it at. Keep `scope` — without it the login is denied.

2. **Open the authorize URL** in a browser (replace the metadata URL, URL-encoded):

    ```
    https://ledger-demo.gluu.info/jans-auth/restv1/authorize?response_type=code&client_id=<URL-ENCODED-METADATA-URL>&redirect_uri=https%3A%2F%2Foidcdebugger.com%2Fdebug&scope=openid&state=t1
    ```

3. **Log in** with your account.

!!! success "Pass"
    You land on `oidcdebugger.com/debug` with a `code=…` in the URL.

!!! failure "Fail"
    An error page, or `error=access_denied` — see the last section.

## Test 2 — ID-JAG issuance

Goal: exchange a user's `id_token` for an ID-JAG assertion. Run these on the server host.

**1. Create a requester client** (confidential, with the token-exchange grant):

```bash
/opt/jans/jans-cli/config-cli.py --operation-id post-oauth-openid-client --data '{
  "displayName": "id-jag-requester",
  "applicationType": "web",
  "grantTypes": ["authorization_code", "urn:ietf:params:oauth:grant-type:token-exchange"],
  "responseTypes": ["code"],
  "tokenEndpointAuthMethod": "client_secret_basic",
  "redirectUris": ["https://oidcdebugger.com/debug"],
  "scopes": ["inum=F0C4,ou=scopes,o=jans"],
  "subjectType": "public"
}' | sed 's/\x1b\[[0-9;]*m//g'
```

Note the `inum` (= `CLIENT_ID`). The returned `clientSecret` is encrypted — get the usable plaintext:

```bash
SECRET=$(/opt/jans/bin/encode.py -decode '<clientSecret from above>')
CID='<inum from above>'
```

**2. Get a user `id_token`.** Open this in a browser, log in, copy the `code`:

```
https://ledger-demo.gluu.info/jans-auth/restv1/authorize?response_type=code&client_id=<CID>&redirect_uri=https%3A%2F%2Foidcdebugger.com%2Fdebug&scope=openid&state=t2
```

Then exchange it (all in one shell so the variables persist):

```bash
ID_TOKEN=$(curl -sk https://ledger-demo.gluu.info/jans-auth/restv1/token \
  -u "$CID:$SECRET" \
  -d grant_type=authorization_code -d code='<CODE>' \
  -d redirect_uri=https://oidcdebugger.com/debug \
  | python3 -c 'import sys,json;print(json.load(sys.stdin)["id_token"])')
echo "len=${#ID_TOKEN}"
```

**3. Mint the ID-JAG:**

```bash
curl -sk https://ledger-demo.gluu.info/jans-auth/restv1/token \
  -u "$CID:$SECRET" \
  -d grant_type=urn:ietf:params:oauth:grant-type:token-exchange \
  -d requested_token_type=urn:ietf:params:oauth:token-type:id-jag \
  -d subject_token="$ID_TOKEN" \
  -d subject_token_type=urn:ietf:params:oauth:token-type:id_token \
  -d audience=https://ledger-demo.gluu.info \
  -d scope=openid | python3 -m json.tool
```

!!! success "Pass"
    The response has `"issued_token_type": "urn:ietf:params:oauth:token-type:id-jag"` and an `access_token` (the ID-JAG JWT). Paste that JWT into [jwt.io](https://jwt.io) — the header shows `typ: oauth-id-jag+jwt`, and the payload carries your `sub`, `client_id`, and `aud`.

## If something fails

| Symptom | Cause | Fix |
| --- | --- | --- |
| `access_denied` after login (Test 1) | Metadata doc has no `scope` | Add `"scope": "openid"` to the metadata doc |
| `access_denied`, and you edited the metadata | Old client cached | Ask us to clear the cached CIMD client, then retry |
| `invalid_client` (Test 2) | Sent the encrypted secret | Decrypt with `encode.py -decode` and use the plaintext |
| `invalid_client`, secret looks right | Wrong shell / empty variable | Run all Test 2 steps in the **same** shell session |
| `invalid_grant` on code exchange | Code expired or reused | Codes last ~1 min and are single-use — get a fresh one |
| Login page never returns a code | Redirect URI mismatch | `redirect_uri` must exactly match the registered one |

Still stuck: send us the `CorrelationId` shown in the error and we will trace it.
