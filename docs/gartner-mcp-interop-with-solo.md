# Gluu + Solo.io for the Gartner Demo: First Draft

*Author: Zico (Gluu). First draft, 2026-10-04. Comments and corrections welcome.*

## 1. Why are we doing this?

AI agents are starting to call tools and data in company systems. They do this through a standard called **MCP** (Model Context Protocol). Companies now ask two simple questions:

- **Which agents are allowed to connect to my systems?**
- **How can an agent from one company use a service in another company, on behalf of a user, without asking the user to log in again?**

The **OpenID Foundation** is running a public test event to show that open standards can answer both questions. Companies test their products together. The results will be presented at the **Gartner Identity & Access Management Summit in Las Vegas, December 7–9, 2026**.

We want to be part of it because:
- it shows Gluu is ready for AI agent security,
- it is free and visible to the whole industry,
- it lets us prove our server works with another vendor's product.

**First deadline: October 16, 2026.** By then we must complete at least one successful test with a partner.

## 2. What will the demo show?

Three things, in simple words:

1. **Agent identity (CIMD).** An agent introduces itself with a web address (a URL) that describes who it is. Our server reads that page and decides whether to let the agent in. No manual registration.
2. **Cross-company access (ID-JAG).** A user logs in at their own company. Their agent then gets access to a service at another company, with no second login. The user's company vouches for them, and the other company's server issues the access.
3. **Protected tools (MCP + OAuth).** The MCP server refuses requests without a valid token, and tells the agent where to get one.

## 3. What is ledger-demo.gluu.info?

It is our test and demo server. It runs **Gluu Flex (Janssen 2.4.1)** on an Ubuntu virtual machine with PostgreSQL.

In this demo it plays the **authorization server**: it checks who is calling and issues access tokens. It can also act as the **identity provider**, which issues the user's "vouching" token (the ID-JAG).

## 4. What is Solo.io?

Solo.io is a company that makes software for running and securing AI systems. The pieces we care about:

- **agentgateway**: a gateway that sits between AI agents and tools (MCP servers). It checks tokens and gets access for the agent.
- **kagent / kmcp**: tools to run AI agents and MCP servers.

Solo has a public demo lab (https://github.com/nickgamb/solo-lab) that already does the cross-company flow (ID-JAG). In their lab, the "other company" server is a Keycloak server. **Our plan is to put Gluu in that spot.**

In the event, Solo would be our **MCP gateway** partner, and we would be the **authorization server**.

## 5. How Solo and Gluu fit together

In simple steps:

1. The user logs in at their company's identity provider.
2. The user's AI agent asks Solo's **agentgateway** to use a tool at another company.
3. agentgateway asks the identity provider for a short "vouching" token (the **ID-JAG**) for that other company.
4. agentgateway sends the ID-JAG to **Gluu (ledger-demo)**.
5. Gluu checks the ID-JAG is real, trusted, and meant for Gluu, and issues an **access token**.
6. agentgateway uses that access token to call the **MCP server**, which only accepts tokens from Gluu.

**Roles:**

| Role | Who |
|---|---|
| Agent gateway | Solo agentgateway |
| Authorization server (receives ID-JAG, issues access token) | **Gluu ledger-demo** |
| Identity provider (issues ID-JAG) | Solo's Keycloak at first. Gluu can do this later, since it already issues ID-JAGs |
| MCP server | Solo's sample server, or a small one of our own |

## 6. What has been done already

On ledger-demo:
- Server installed and running with a real **Let's Encrypt certificate** (auto-renews).
- **CIMD works.** We found and patched two bugs, and the login flow with a CIMD client now returns an authorization code.
- **ID-JAG issuance works.** The server can create a valid ID-JAG token (signed, 300 seconds lifetime).
- **Passkey login** and **self-registration** are turned on, so demo users can be created quickly.
- A short **customer test guide** is published at github.com/mzico/ledger-demo.
- The disk-full problem was fixed (old temporary files had filled the disk).

For the project:
- We read the OpenID test plan and the Solo lab, and we know what each side must do.

## 7. What is left to configure

**On ledger-demo (Gluu side):**
- [ ] Advertise **PKCE S256** in the server's public settings (it is required by the test and missing today).
- [ ] Add the standard `/.well-known/oauth-authorization-server` address (it returns "not found" today).
- [ ] Check that **CIMD works at the token step**, not just at login.
- [ ] **Receive and accept an ID-JAG from another identity provider** (trust their issuer, map the user). We have only tested creating one so far.
- [ ] Create a **demo user** that matches the user in the partner's identity provider.
- [ ] Register Solo's gateway as a client (preferably with a key, not a password).
- [ ] Cleanup job for the temporary files, and set logging back from DEBUG.
- [ ] Rotate or delete the test client secret after demos.

**Together with Solo:**
- [ ] Agree on the test (who is client, who is identity provider, who is MCP server).
- [ ] Run a first test with a hand-made ID-JAG, then with Solo's gateway.
- [ ] Set up an MCP server that accepts Gluu tokens.
- [ ] Save logs and fill in the OpenID result tables. Both sides must agree before submitting.
- [ ] Plan and rehearse the live demo for December.

## 8. Rough timeline

| When | What |
|---|---|
| Now | Contact Solo, confirm we are registered with the OpenID group, fix the small Gluu items |
| By Oct 16 | First successful test with Solo (ID-JAG into Gluu) |
| Oct–Nov | More tests (CIMD, MCP server), collect logs |
| Before Dec 7 | Submit results, rehearse the demo |
| Dec 7–9 | Gartner summit in Las Vegas |

## 9. Questions we still need answered

**For Solo:**
- Are you taking part in the event? In which roles?
- Can your gateway identify itself with a CIMD URL and sign requests with a key?
- Which identity provider will you use to create the ID-JAG?
- Will the demo run live from your lab, or from a hosted setup that can reach our public server?

**For the Gluu team:**
- Who owns the remaining fixes and the final testing?
- Who attends the weekly OpenID calls (Mondays, 11 AM Pacific)?
- Do we want Gluu to also act as the identity provider in the demo?

## 10. Links

- OpenID event: https://openid.net/call-for-participation-demonstrate-mcp-based-ai-agent-security-with-open-identity-standards-2/
- Solo lab: https://github.com/nickgamb/solo-lab
- Our test guide: https://github.com/mzico/ledger-demo
