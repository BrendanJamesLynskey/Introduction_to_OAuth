# Introduction to OAuth

**The delegated-authorisation framework that runs the modern web**

*Purpose · History · OAuth 1.0 → 2.0 → 2.1 — flows, tokens, scopes, PKCE, OIDC. Companion deck covers MCP & provider landscape.*

```
User -> Client -> Authorization Server -> Token -> Resource Server
```

Delegate  |  Authorise  |  Issue  |  Verify

---

## Table of Contents

1. [Topics](#slide-01--topics)
2. [Why OAuth Exists — The Purpose](#slide-02--why-oauth-exists--the-purpose)
3. [A Brief History](#slide-03--a-brief-history)
4. [OAuth 1.0a — Signatures & the Three-Legged Dance](#slide-04--oauth-10a--signatures--the-three-legged-dance)
5. [OAuth 2.0 — The Framework Reset](#slide-05--oauth-20--the-framework-reset)
6. [Roles & Terminology](#slide-06--roles--terminology)
7. [Tokens — Three Different Jobs](#slide-07--tokens--three-different-jobs)
8. [Authorization Code Flow](#slide-08--authorization-code-flow)
9. [PKCE — Proof Key for Code Exchange](#slide-09--pkce--proof-key-for-code-exchange)
10. [The Deprecated Flows — What Not To Use](#slide-10--the-deprecated-flows--what-not-to-use)
11. [Client Credentials — Machine-to-Machine](#slide-11--client-credentials--machine-to-machine)
12. [Device Authorization Grant](#slide-12--device-authorization-grant)
13. [Scopes, Audience & Resource Indicators](#slide-13--scopes-audience--resource-indicators)
14. [OpenID Connect — Identity on Top](#slide-14--openid-connect--identity-on-top)
15. [JWTs in OAuth — Anatomy & Verification](#slide-15--jwts-in-oauth--anatomy--verification)
16. [OAuth 2.1 — The Consolidation](#slide-16--oauth-21--the-consolidation)
17. [DPoP & mTLS — Beyond Bearer](#slide-17--dpop--mtls--beyond-bearer)
18. [Common Attacks & How OAuth Defends](#slide-18--common-attacks--how-oauth-defends)
19. [OAuth vs SAML vs API Keys vs WebAuthn](#slide-19--oauth-vs-saml-vs-api-keys-vs-webauthn)
20. [Real-World Providers — Quick Tour](#slide-20--real-world-providers--quick-tour)
21. [Libraries — Don't Hand-Roll This](#slide-21--libraries--dont-hand-roll-this)
22. [Choosing a Flow — Decision Matrix](#slide-22--choosing-a-flow--decision-matrix)
23. [Cheat Sheet — Endpoints, Params, Errors](#slide-23--cheat-sheet--endpoints-params-errors)
24. [Gotchas Caught in Production](#slide-24--gotchas-caught-in-production)
25. [Summary & Where to Next](#slide-25--summary--where-to-next)

---

## Slide 01 — Topics

### Foundations

- The problem OAuth solves — and what came before
- OAuth 1.0a — signatures, nonces, three-legged dance
- OAuth 2.0 — RFC 6749, the framework reset
- Roles, tokens, endpoints, terminology

### Flows

- Authorization Code (the only one you should use)
- PKCE — RFC 7636, mandatory in 2.1
- Client Credentials & Device Code grants
- Implicit & Password grants — and why they're dead

### Tokens & identity

- Access, refresh, ID tokens — three different jobs
- Scopes, audience, resource indicators
- Bearer vs DPoP vs mTLS sender-constrained
- OpenID Connect — the identity layer on top

### Modern & security

- OAuth 2.1 — what's in, what's out
- Common attacks: redirect, CSRF, mix-up, theft
- OAuth vs SAML vs API keys vs WebAuthn
- Choosing a flow & pointer to Part 2 (MCP & providers)

---

## Slide 02 — Why OAuth Exists — The Purpose

OAuth solves one specific problem: **letting an application act on a user's behalf at a third-party service, without the user handing over their password**. It is an *authorisation* framework — not, by itself, an authentication protocol.

### The "password anti-pattern" it killed

- 2006: a printing app wants your Flickr photos. You paste your Flickr password into it.
- The app now has full account access — forever — and there is no way to revoke just *that* app.
- Users were trained to type credentials into anything that asked. Phishing was indistinguishable from legitimate use.
- Password resets at one site cascaded across every integration.

### What OAuth actually delegates

- **Scoped** access — "read photos", not "everything"
- **Time-bounded** — short-lived access tokens
- **Revocable per-app** — without rotating your password
- **Auditable** — the AS knows which client did what

### Three things OAuth is *not*

- **Not authentication** — it doesn't tell you "who" the user is. (OIDC adds that on top.)
- **Not encryption** — TLS does that. OAuth assumes channels are already secure.
- **Not a single protocol** — it's a framework of grant types you assemble.

> "Application **A** wants to access resource **R** belonging to user **U**, hosted at server **S**, without ever seeing **U**'s password."

Every grant type, token format, and security mechanism in OAuth exists to safely accomplish that one sentence.

---

## Slide 03 — A Brief History

```
2006 → 2007 → 2010 → 2012 → 2014 → 2020 → 2024+
 │      │      │      │      │      │       │
 │      │      │      │      │      │       └─ DPoP, RAR, MCP profile
 │      │      │      │      │      └─ OAuth 2.1 draft + Sec BCP 240
 │      │      │      │      └─ OpenID Connect 1.0
 │      │      │      └─ OAuth 2.0 — RFC 6749
 │      │      └─ OAuth 1.0a — RFC 5849
 │      └─ OAuth 1.0 draft
 └─ Twitter delegation problem
```

Eighteen years from "the password anti-pattern" to a sender-constrained, audience-bound, PKCE-mandatory OAuth 2.1.

### The triggers (2006–2007)

- Blaine Cook (Twitter) needed delegation for OpenID-based Twitter signups.
- Joined by Chris Messina, Larry Halff, David Recordon and the Ma.gnolia / Pownce crowd.
- OAuth 1.0 published Dec 2007, refined to 1.0a in 2009 after a session-fixation flaw.
- Standardised by IETF as **RFC 5849** in April 2010.

### The 2.0 reset and beyond

- 2010: Eran Hammer chairs OAuth WG, drives a new charter — bearer tokens, no signatures, separate flows for native & web clients.
- 2012: **RFC 6749** + **RFC 6750** ship. Hammer publicly resigns over its complexity.
- 2014: OpenID Connect 1.0 finalised — identity layer on top of OAuth 2.0.
- 2020+: BCP 240 (Security Best Current Practice) + OAuth 2.1 draft consolidate ten years of lessons.

---

## Slide 04 — OAuth 1.0a — Signatures & the Three-Legged Dance

OAuth 1.0a was a **single, self-contained protocol**. Every request was signed with HMAC-SHA1 (or RSA-SHA1) using a per-client consumer secret plus a per-user token secret. No TLS required — the signature protected the request itself.

### The three legs

1. **Request token** — client → service, gets a temporary token.
2. **User authorisation** — user redirected to service, approves, gets a verifier code.
3. **Access token exchange** — client swaps request token + verifier for a long-lived access token + secret.

### A signed request

```
GET /api/photos HTTP/1.1
Host: api.example.com
Authorization: OAuth
  oauth_consumer_key="abc123",
  oauth_token="xyz789",
  oauth_signature_method="HMAC-SHA1",
  oauth_timestamp="1700000000",
  oauth_nonce="r4nd0m",
  oauth_version="1.0",
  oauth_signature="kxQ8...%3D"
```

The signature covers method + URL + every parameter. Replay protection comes from the timestamp + nonce.

### Why it was abandoned

- Signing every request was hard to get right — string normalisation, percent-encoding edge cases.
- Painful in mobile apps and JavaScript clients (no easy crypto).
- Tokens were long-lived bearer-equivalents anyway, once stolen.
- Each language had a slightly broken library.

### Where you'll still see it

X / Twitter API v1.1, some legacy enterprise APIs, and Magento. Everything new uses OAuth 2.0.

---

## Slide 05 — OAuth 2.0 — The Framework Reset

OAuth 2.0 (**RFC 6749**, October 2012) is deliberately a **framework, not a protocol**. It defines roles, endpoints, and a small set of grant types — the rest is filled in by companion RFCs and profiles.

### Big design changes vs 1.0a

- **No request signing** — bearer tokens over HTTPS instead.
- **Multiple grant types** for different client environments (web, native, M2M, devices without browsers).
- **Short-lived access tokens** + long-lived refresh tokens — limits damage of theft.
- **JSON** everywhere — no XML or form-encoded signatures.

### The four canonical grant types

| Grant | For | Status today |
|---|---|---|
| Authorization Code | Server & native apps with a user | **Recommended** |
| Implicit | JS in the browser, pre-PKCE | **Removed in 2.1** |
| Resource Owner Password | First-party "trusted" apps | **Removed in 2.1** |
| Client Credentials | Machine-to-machine (no user) | **Recommended** |

### Famous criticism

> "OAuth 2.0 is bad, complicated, and fundamentally insecure." — **Eran Hammer**, *"OAuth 2.0 and the Road to Hell"*, July 2012.

He stepped down as lead author and removed his name from the spec. His point: by removing signatures and shipping an extensible framework, the WG pushed security decisions onto every implementer — and most got them wrong.

### Plus, added later as separate RFCs

- **RFC 7636** PKCE · **RFC 8628** Device Authorization · **RFC 8414** AS Metadata
- **RFC 7591** Dynamic Client Registration · **RFC 8707** Resource Indicators
- **RFC 9449** DPoP · **RFC 9396** Rich Authorization Requests

---

## Slide 06 — Roles & Terminology

```
[Resource Owner] — [Client] — [Authorization Server] — [Resource Server]
   the user        the app      issues tokens             verifies tokens
```

AS & RS may be the same product (Google) or completely separate services (your API + Auth0).

### Endpoints on the AS

- `/authorize` — user-facing, where consent happens
- `/token` — back-channel, exchanges codes/refresh for access tokens
- `/jwks` — public keys to verify signed tokens
- `/.well-known/oauth-authorization-server` — RFC 8414 metadata
- `/revoke`, `/introspect`, `/userinfo` (OIDC)

### Client types

- **Confidential** — can keep a secret (server-side web app, backend service).
- **Public** — cannot (SPA, mobile, CLI, desktop). Uses PKCE instead of a client secret.

### Other terms you'll see

**Scope** — string identifying a permission. **Audience** — which RS the token is for (RFC 8707). **Claims** — assertions inside a token. **Consent screen** — the AS UI users see.

---

## Slide 07 — Tokens — Three Different Jobs

| Token | What it represents | Lifetime | Sent to | Format |
|---|---|---|---|---|
| **Access token** | Permission to call the resource server | 5 min – 1 hour | Resource Server | Opaque or JWT |
| **Refresh token** | Permission to mint new access tokens | Hours – months | Authorization Server only | Opaque (always) |
| **ID token** | Statement about the user (OIDC) | 5–15 min | Stays with the client | Signed JWT (always) |

### Bearer tokens (RFC 6750)

```
GET /api/me HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJSUzI1NiI...
```

"Bearer" = whoever holds it can use it. **Steal the token → impersonate the user** until it expires.

### JWT vs opaque

- **Opaque** — random string. Resource server must call `/introspect` on the AS to validate.
- **JWT** — self-contained, signed. RS verifies offline using the AS's JWKS public keys.
- JWT scales better but is harder to revoke before expiry.

### Sender-constrained tokens — beyond bearer

- **DPoP** (RFC 9449) — client proves possession of a key on every request. Token + JWS proof.
- **mTLS** (RFC 8705) — token bound to the client TLS certificate.
- Either one defeats stolen-token replay. Required by FAPI 2.0; recommended for high-value APIs and remote MCP servers.

### Refresh token rotation

Each `/token` call with `grant_type=refresh_token` issues a *new* refresh token and invalidates the old one. If two clients ever present the same refresh token, the AS treats it as theft and revokes the whole chain. Mandatory for public clients in 2.1.

---

## Slide 08 — Authorization Code Flow

The default OAuth flow for *any* app where a user is present. Everything else is a deprecated shortcut.

```
User-Agent       Client          AuthZ Server         Resource Server
    │              │                  │                     │
1.  ├─click "log in"─▶               │                     │
2.  ◀──302 → /authorize?…&code_challenge=…─                │
3.  ├──GET /authorize → user logs in & consents──▶          │
4.  ◀──302 redirect_uri?code=AUTH_CODE&state=…─            │
5.  ├──delivers ?code=…──▶                                  │
6.                ├──POST /token (code, code_verifier, secret)─▶
7.                ◀──{ access_token, refresh_token, id_token }─
8.                ├──GET /api/me  Authorization: Bearer …──▶
9.                ◀──200 { user data }──
```

### Front vs back channel

- **Front-channel** — steps 2–5 go through the user's browser. Anything visible — including the code — must be considered intercept-able.
- **Back-channel** — step 6 is server-to-server, TLS, with PKCE proof + (for confidential clients) a client secret. This is where the real token lives.
- **Why the dance?** — splitting front and back channels means a leaked code alone is useless without the back-channel verifier & secret.

---

## Slide 09 — PKCE — Proof Key for Code Exchange

**RFC 7636** (2015). Originally a fix for native apps that couldn't keep a client secret. **In OAuth 2.1, PKCE is mandatory for every client** — public or confidential.

### The mechanism

```python
# Client generates a high-entropy random string
code_verifier = base64url(random(32))           # ~43 chars

# And derives the challenge
code_challenge = base64url(SHA-256(code_verifier))

# Sent on /authorize  (front-channel, visible)
GET /authorize?
    response_type=code&
    client_id=app123&
    code_challenge=XYZ&
    code_challenge_method=S256&
    redirect_uri=...&state=...

# Sent on /token       (back-channel, secret)
POST /token  code=AUTH_CODE&
             code_verifier=ORIGINAL_VERIFIER&
             ...
```

### Why it works

- The AS stores the `code_challenge` alongside the issued `code`.
- At `/token`, the AS computes `SHA-256(code_verifier)` and compares.
- An attacker who steals the front-channel code **cannot redeem it** — they don't have the verifier.
- An attacker who steals only the back-channel verifier has nothing to redeem either.

### Mandatory in OAuth 2.1

- Always use `S256`. `plain` is now forbidden.
- Verifier must be 43–128 URL-safe characters.
- Even confidential clients must use it — defence in depth.

### Common mistake

Hardcoding a per-app verifier instead of generating fresh per-request. PKCE is *per-authorisation* — re-use defeats it.

---

## Slide 10 — The Deprecated Flows — What Not To Use

### Implicit Flow (`response_type=token`)

- Designed for SPAs in 2012 — token returned *directly* in the redirect URL fragment.
- No back-channel exchange, no client secret, no PKCE.
- Tokens leaked via browser history, referer headers, server logs.
- **Removed** in OAuth 2.1. SPAs now use Authorization Code + PKCE.

```
# Old, dangerous
GET /authorize?response_type=token&client_id=...
# returns: https://app/cb#access_token=eyJ...&token_type=Bearer

# Modern replacement
GET /authorize?response_type=code&code_challenge=...&...
```

### Resource Owner Password Credentials

- User gives the client their username + password directly.
- Reintroduces the exact anti-pattern OAuth was designed to kill.
- Cannot do MFA, can't show consent, can't be revoked at the AS.
- **Removed** in OAuth 2.1. Use Auth Code + PKCE in a system browser instead.

```
POST /token
grant_type=password&
username=alice&password=hunter2&
client_id=...      # never do this
```

### "But I trust my own app!"

That argument was used by every breached first-party app. Modern guidance (BCP 240): always launch a real browser, even for first-party apps. AppAuth libraries (iOS / Android) make this painless.

---

## Slide 11 — Client Credentials — Machine-to-Machine

For service-to-service calls where **no user is involved**. The client authenticates with its own credentials and gets an access token bound to itself, not a user.

```
POST /token HTTP/1.1
Host: auth.example.com
Authorization: Basic base64(client_id:client_secret)
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&
scope=invoices:read&
audience=https://api.billing.internal

# response
{ "access_token": "eyJ...",
  "token_type": "Bearer",
  "expires_in": 3600 }
```

### Use cases

- Cron jobs hitting an internal API
- Microservices behind an internal gateway
- CI pipelines pushing artefacts
- Backend MCP servers calling other backend APIs (no user context)

### Stronger client auth

- **`private_key_jwt`** — client signs a JWT assertion with its own key (RFC 7523). No shared secret in transit.
- **`tls_client_auth`** — mTLS with a client certificate (RFC 8705).
- Both required by FAPI 2.0; preferred over `client_secret_basic` for sensitive M2M.

### Common misuse

Using client credentials on behalf of a user — the resulting token has no user identity, only "client X's token". Auditing breaks. Use Auth Code or token exchange instead.

---

## Slide 12 — Device Authorization Grant

**RFC 8628** (2019). For devices with no browser or limited input — TVs, CLIs, IoT, terminal sessions. The flow you've used a hundred times: *"go to `github.com/login/device` and enter ABCD-1234"*.

```
Device (CLI/TV)        Auth Server          User on Phone
      │                      │                     │
1.    ├──POST /device_authorization──▶             │
2.    ◀──{ device_code, user_code, verification_uri }─
      │   show "ABCD-1234"                         │
3.    │                      ◀──user types code, signs in─
4.    ├──POST /token (poll)──▶                     │
        → authorization_pending → eventually access_token
```

### Where you've seen it

- `gh auth login`, `gcloud auth login`
- Smart TVs signing into Netflix / YouTube / Disney+
- Microsoft Authenticator on the phone for Office logins
- Many MCP server first-time setups for headless agents

### Security notes

- The user code is short (8 chars) — rate-limit verification attempts.
- Phishable: an attacker can ask *you* to type their code into the AS. Show what scopes & client are being approved.
- Pair with PKCE-equivalent device-code-binding where possible.

---

## Slide 13 — Scopes, Audience & Resource Indicators

Scope is the OAuth answer to "*what can this token do?*". Audience is "*which API can it do it to?*". Both must be checked.

### Scopes — convention, not rule

```
# Google
scope=openid email profile
      https://www.googleapis.com/auth/drive.readonly

# GitHub
scope=repo:status read:user user:email

# Generic verb:resource
scope=invoices:read invoices:write
      payments:refund

# OIDC reserved
scope=openid          # required for OIDC
scope=offline_access  # request a refresh token
```

Strings are opaque to OAuth. Each AS / API defines its own vocabulary. Granularity is a UX vs security trade-off.

### Resource Indicators — RFC 8707

```
POST /token
grant_type=authorization_code&
code=...&
resource=https://api.invoices.example.com&
resource=https://api.payments.example.com
```

The `resource` parameter pins the issued token's `aud` claim. The RS must reject any token whose `aud` doesn't match its own URL.

### Why audience matters — confused deputy

Without audience binding, a token issued for a friendly RS can be replayed against an unrelated RS that trusts the same AS. This is the root cause of the "token passthrough" vulnerability you'll see again in Part 2 (MCP).

### Consent UX

"*App X wants to: read your invoices · refund payments*" — the consent screen renders scope IDs into human strings the AS controls. Bad scope names = bad consent UX.

---

## Slide 14 — OpenID Connect — Identity on Top

OAuth 2.0 says *nothing* about who the user is. **OpenID Connect (OIDC)** — finalised 2014 by the OpenID Foundation — is a thin identity layer on top of OAuth 2.0. It is what powers "Sign in with Google / Microsoft / Apple".

### What it adds

- **ID token** — a signed JWT about the user, returned alongside the access token.
- **`/userinfo`** endpoint — fetch user attributes with the access token.
- **`openid` scope** — required to opt in.
- **Discovery** — `/.well-known/openid-configuration` JSON document.
- **Standard claims** — `sub`, `email`, `name`, `picture`, etc.

### A typical ID token (decoded)

```json
{
  "iss": "https://accounts.google.com",
  "sub": "1098765432109876543210",
  "aud": "client123.apps.googleusercontent.com",
  "iat": 1700000000,
  "exp": 1700003600,
  "email": "alice@example.com",
  "email_verified": true,
  "name": "Alice Example",
  "picture": "https://lh3..."
}
```

### OAuth vs OIDC — the rule

- "**Log in** with X" → OIDC.
- "**Connect** X to access something" → plain OAuth.
- If you want both — request `openid` + your API scopes in the same flow.

### Discovery example

```
$ curl https://accounts.google.com/.well-known/openid-configuration
{
  "issuer": "https://accounts.google.com",
  "authorization_endpoint": "https://accounts.google.com/o/oauth2/v2/auth",
  "token_endpoint": "https://oauth2.googleapis.com/token",
  "jwks_uri": "https://www.googleapis.com/oauth2/v3/certs",
  "userinfo_endpoint": "https://openidconnect.googleapis.com/v1/userinfo",
  ...
}
```

### Don't use access tokens for identity

An access token says *"this client may call this API"*, not *"this is user U"*. Always derive identity from the ID token's `sub` claim, never by introspecting an access token.

---

## Slide 15 — JWTs in OAuth — Anatomy & Verification

### Three dot-separated parts

```
eyJhbGciOiJSUzI1NiIsImtpZCI6IjE0MyJ9        # header
.eyJpc3MiOiJodHRwczovL2lzcy5leGFtcGxlIiw   # payload
  ic3ViIjoiNDQiLCJhdWQiOiJhcGkiLCJleHAiOj
  E3MDAwMDM2MDB9
.kxQ8...                                    # signature

# Header
{ "alg":"RS256", "kid":"143", "typ":"JWT" }

# Payload
{ "iss":"https://iss.example",
  "sub":"44", "aud":"api",
  "iat":1700000000, "exp":1700003600,
  "scope":"invoices:read",
  "client_id":"app123" }
```

### Validation checklist

1. Verify `alg` is what you expect (RS256, ES256). Reject `none`; reject HS256 if you only have a public key.
2. Find the public key in JWKS by `kid`. Cache, but rotate.
3. Verify the signature.
4. Check `iss` matches the AS you trust.
5. Check `aud` contains your RS identifier.
6. Check `exp` is in the future and `nbf` in the past.
7. Check `scope` covers the operation requested.

### Classic JWT mistakes

- Trusting `alg:none` tokens.
- HS256/RS256 confusion — verifying an attacker-supplied HS256 token using the RSA public key as the HMAC secret.
- Skipping `aud` check (confused deputy).
- Treating an unsigned base64 payload as authoritative.
- Putting refresh tokens or PII in JWTs that hit logs.

---

## Slide 16 — OAuth 2.1 — The Consolidation

OAuth 2.1 (**`draft-ietf-oauth-v2-1`**, in late-stage IETF review) is not a new protocol — it's RFC 6749 + a decade of errata + the security best practices (BCP 240) folded into one document.

### What's in

- **Authorization Code + PKCE for everyone** (public & confidential).
- **Refresh token rotation** required for public clients.
- Strict **`redirect_uri`** string matching — no wildcards.
- HTTPS everywhere.
- Client Credentials and Device Code retained.

### What's out

- **Implicit Flow** — gone.
- **Resource Owner Password Credentials** — gone.
- **Bearer tokens in URI query strings** — banned.
- **PKCE `plain` method** — banned (S256 only).

### Companion specs you should also know

| RFC | What it adds |
|---|---|
| RFC 8252 | OAuth for Native Apps (use the system browser) |
| RFC 8414 | Authorization Server Metadata (`/.well-known/...`) |
| RFC 7591 / 7592 | Dynamic Client Registration / Management |
| RFC 8628 | Device Authorization Grant |
| RFC 8693 | Token Exchange |
| RFC 8705 | mTLS Client Auth & Sender-Constrained Tokens |
| RFC 8707 | Resource Indicators (audience binding) |
| RFC 9068 | JWT Profile for Access Tokens |
| RFC 9126 | Pushed Authorization Requests (PAR) |
| RFC 9396 | Rich Authorization Requests (RAR) |
| RFC 9449 | DPoP — sender-constrained bearer tokens |
| RFC 9728 | OAuth 2.0 Protected Resource Metadata |
| BCP 240 | OAuth 2.0 Security Best Current Practice |

---

## Slide 17 — DPoP & mTLS — Beyond Bearer

Bearer tokens have one fatal property: anyone who steals one can use it. **Sender-constrained** tokens fix that by binding the token to a key only the legitimate client controls.

### DPoP — Demonstration of Proof-of-Possession

```
POST /api/payment HTTP/1.1
Host: api.example.com
Authorization: DPoP eyJhbGciOiJSUzI1NiI...
DPoP: eyJ0eXAiOiJkcG9wK2p3dCIsImFsZyI6IkVTMjU2In0.
      eyJqdGkiOiJlMS4uIiwiaHRtIjoiUE9TVCIsImh0dSI6
       Imh0dHBzOi8vYXBpLmV4YW1wbGUuY29tL3BheW1lbnQiLA
       AiaWF0IjoxNzAwMDAwMDAwfQ.signature
```

- Client mints a fresh JWT (the DPoP proof) per request, signed with a private key it generated locally.
- Proof binds the request to: HTTP method, URL, timestamp, and a unique `jti`.
- Token's `cnf.jkt` claim contains a hash of the public key — RS rejects mismatched proofs.

### mTLS sender-constrained tokens (RFC 8705)

- Client establishes a mutual-TLS connection with its X.509 certificate.
- AS records a hash of the client cert in the token's `cnf.x5t#S256` claim.
- RS verifies the calling cert hash matches.
- Standard in banking (FAPI 2.0), some healthcare deployments.

### When to use which

|  | DPoP | mTLS |
|---|---|---|
| Browser clients | yes | no |
| Mobile clients | yes | painful |
| Backend services | fine | preferred |
| Infra burden | low | PKI required |

### Why this matters for MCP

Remote MCP servers exchange tokens over the public internet — Bearer tokens here are particularly tempting targets. The MCP authorisation profile recommends DPoP; we'll see this again in Part 2.

---

## Slide 18 — Common Attacks & How OAuth Defends

| Attack | How it works | Defence |
|---|---|---|
| Authorization Code Interception | Native app's custom URI scheme is registered by a malicious app on the same device; it receives the `code`. | **PKCE** — stolen code can't be redeemed without the verifier. |
| CSRF on `redirect_uri` | Attacker tricks victim's browser into completing a flow against the attacker's account. | **`state`** parameter — random, bound to the user's session, validated on callback. |
| Open Redirector | A loose `redirect_uri` match (wildcards) lets an attacker exfiltrate the code via a controlled domain. | **Strict string match** on `redirect_uri` (mandatory in 2.1). |
| Mix-up Attack | Client supports multiple AS; attacker tricks it into sending an honest AS's code to a malicious AS. | **`iss`** parameter on the callback (RFC 9207). Match against the AS the flow started with. |
| Token Theft / Replay | Bearer access token leaks via logs, proxy, browser extension; attacker replays. | **DPoP / mTLS** sender-constrained tokens; short TTLs; refresh rotation. |
| Refresh Token Theft | Long-lived RT exfiltrated from a public client. | **Rotation** — old RT invalidated on every use; reuse detected as compromise. |
| Confused Deputy | Token issued for API A is replayed against API B that also trusts the AS. | **Audience binding** via Resource Indicators (RFC 8707) + `aud` validation. |
| AS Phishing | Attacker hosts a fake consent screen that looks like Google's. | Always launch the AS in the system browser (RFC 8252), not an embedded webview. |
| Click-Through Consent | Users approve everything without reading. | Granular scopes, plain-English consent strings, periodic re-consent. |

### Read this once

RFC 9700 (BCP 240) — *"OAuth 2.0 Security Best Current Practice"*. Forty pages. Every implementer should have read it.

---

## Slide 19 — OAuth vs SAML vs API Keys vs WebAuthn

| Mechanism | What it does | Best for | Caveat |
|---|---|---|---|
| **OAuth 2.0/2.1** | Delegated authorisation (token issuance for third-party apps) | API access on behalf of a user; M2M tokens; mobile/SPA | It's a framework, not a turnkey protocol — easy to misconfigure |
| **OpenID Connect** | Identity / SSO layer on top of OAuth 2.0 | "Sign in with..." flows, modern federated SSO | Adds JWT validation complexity |
| **SAML 2.0** | XML-based browser SSO between IdP and SP | Enterprise SSO (Workday, Salesforce, internal apps) | XML-DSig is brittle; no native API delegation |
| **API keys** | Static shared secret per client | Simple machine-to-machine, internal services, getting started | No expiry, no scopes, painful rotation, no user context |
| **HTTP Basic / Digest** | Username + password per request | Internal admin tools behind a VPN | Sends credentials on every call; no MFA |
| **WebAuthn / Passkeys** | Phishing-resistant user authentication using device-bound keys | First-factor user login; replaces passwords | Authentication only — pair with OIDC for SSO + OAuth for API delegation |
| **Macaroons / Biscuit** | Capability tokens with caveats (attenuation in the client) | Fine-grained delegation chains, offline-friendly | Niche; little ecosystem support |

### Common stacks today

- **Consumer apps** — OIDC for login + OAuth 2.1 + PKCE for API access. Passkeys as the user's first factor.
- **Enterprise SaaS** — SAML to the IdP for SSO + SCIM for provisioning + OIDC for newer apps.
- **Microservices** — OAuth Client Credentials + mTLS sender-constrained tokens; sometimes SPIFFE for workload identity.

### Decision rule of thumb

- "User logs into our app" → OIDC.
- "User authorises another app to call our API" → OAuth Auth Code + PKCE.
- "Service A calls service B with no user" → OAuth Client Credentials (or mTLS).
- "Legacy enterprise customer wants SSO" → SAML.
- "Internal tool, behind VPN, low value" → Maybe API key — but plan migration.

---

## Slide 20 — Real-World Providers — Quick Tour

Every modern login button is OAuth 2.0 + OIDC under the hood. The endpoints differ; the flow is the same.

### Google (Sign in with Google)

```
Discovery:  https://accounts.google.com/.well-known/openid-configuration
Authorize:  https://accounts.google.com/o/oauth2/v2/auth
Token:      https://oauth2.googleapis.com/token
JWKS:       https://www.googleapis.com/oauth2/v3/certs
Scopes:     openid email profile + Google API scopes
```

### GitHub

```
Authorize:  https://github.com/login/oauth/authorize
Token:      https://github.com/login/oauth/access_token
Device:     https://github.com/login/device/code
User:       https://api.github.com/user
Scopes:     repo, read:user, user:email, ...
Note: opaque tokens (not JWTs); separate fine-grained PATs.
```

### Microsoft Entra ID (formerly Azure AD)

```
Discovery:  https://login.microsoftonline.com/{tenant}/v2.0/.well-known/openid-configuration
Authorize:  .../oauth2/v2.0/authorize
Token:      .../oauth2/v2.0/token
Scopes:     openid offline_access User.Read https://graph.microsoft.com/.default
Multi-tenant via {tenant}=common or {tenant}=organizations
```

### "Sign in with Apple"

- OIDC — issues an ID token JWT.
- Quirk: only returns the user's name on *first* login. Lose it and it's gone.
- Required by App Store guidelines if you offer any other social login.

*Part 2 of this deck covers the full provider landscape — Auth0, Okta, AWS Cognito, Stytch, WorkOS, Clerk, Descope, plus the open-source side: Keycloak, Authentik, ZITADEL, Ory Hydra, Authelia, Logto, SuperTokens — and how to wire each into a remote MCP server.*

---

## Slide 21 — Libraries — Don't Hand-Roll This

OAuth is a "*15 lines of code*" demo and a "*thousand-line spec to get right*" reality. Use a library that already passes the conformance suite.

### Client side (your app calls an OAuth-protected API)

- **JS / browser** — `oauth4webapi`, `oidc-client-ts`, Auth0 SPA SDK
- **Node.js** — `openid-client`, `passport-oauth2`
- **Python** — `authlib`, `requests-oauthlib`
- **Go** — `golang.org/x/oauth2` + `github.com/coreos/go-oidc`
- **Rust** — `oauth2` crate, `openidconnect`
- **Java/Kotlin** — Spring Security OAuth Client, Nimbus
- **Mobile** — AppAuth (iOS/Android)

### Resource server side (validate incoming tokens)

- **Node.js** — `jose`, `express-oauth2-jwt-bearer`
- **Python** — `authlib` ResourceProtector, `PyJWT` + JWKS cache
- **Go** — `github.com/golang-jwt/jwt` + `github.com/MicahParks/keyfunc`
- **Rust** — `jsonwebtoken`, `axum-jwks`
- **Java** — Spring Security Resource Server, Nimbus

### Conformance suites worth knowing

- **OpenID Foundation Conformance** — `openid.net/certification` — the gold standard.
- **FAPI conformance** for high-security profiles.
- Use `oauth-tools` / `oauth-debugger` in dev to inspect what flows your AS actually does.

---

## Slide 22 — Choosing a Flow — Decision Matrix

| Your client is... | User present? | Use this flow | Notes |
|---|---|---|---|
| Server-side web app (Express, Rails, Django) | yes | **Authorization Code + PKCE** | Confidential client; can keep a client secret. |
| SPA (React, Vue, Angular in the browser) | yes | **Authorization Code + PKCE** | Public client; no secret. Or use a backend-for-frontend. |
| Mobile app (iOS, Android) | yes | **Authorization Code + PKCE** | System browser via AppAuth; never an embedded webview (RFC 8252). |
| CLI / desktop tool | yes | **Auth Code + PKCE** (loopback) *or* **Device Code** | Loopback works on dev machines; device code is best when no browser is local. |
| Smart TV / IoT / kiosk | yes (on a phone) | **Device Authorization** | RFC 8628. |
| Cron job / backend service | no | **Client Credentials** | Use `private_key_jwt` or mTLS for stronger client auth. |
| Service-A calling Service-B with user context | indirectly | **Token Exchange** (RFC 8693) | Preserves the original user identity through the chain. |
| Local MCP server (stdio) | n/a | **no OAuth** | Process boundary is local; trust the launching app. |
| Remote MCP server (HTTP) | yes | **Auth Code + PKCE + Resource Indicators (+ DPoP)** | See Part 2 of this deck. |

---

## Slide 23 — Cheat Sheet — Endpoints, Params, Errors

### `/authorize` parameters

| Param | Required | Notes |
|---|---|---|
| `response_type` | yes | Always `code` |
| `client_id` | yes | Issued by AS |
| `redirect_uri` | yes | Pre-registered, exact match |
| `scope` | yes | Space-separated |
| `state` | CSRF | Random per-request |
| `code_challenge` | PKCE | S256 hash |
| `code_challenge_method` | yes | `S256` |
| `nonce` | OIDC | Reflected in ID token |
| `resource` | RFC 8707 | Pin token audience |

### `/token` request

```
POST /token HTTP/1.1
Authorization: Basic base64(client_id:secret)   # confidential
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&
code=AUTH_CODE&
redirect_uri=https://app/cb&
code_verifier=ORIGINAL_VERIFIER
```

### `/token` error responses

- `invalid_request` — missing/bad parameter
- `invalid_client` — auth failed
- `invalid_grant` — code/refresh token rejected
- `unauthorized_client` — client not allowed this grant
- `unsupported_grant_type`
- `invalid_scope`
- `authorization_pending` / `slow_down` — device code polling

### RS error (RFC 6750)

```
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer realm="api",
   error="invalid_token",
   error_description="The access token expired"
```

---

## Slide 24 — Gotchas Caught in Production

### Client / browser side

- **No `state`** → CSRF on the callback. Pick a UUID per flow, store in session, compare.
- **Trusting the access token for identity** → use the ID token's `sub`.
- **Storing tokens in `localStorage`** → XSS exfiltrates them. Prefer HttpOnly cookies + a backend-for-frontend.
- **Embedded WebView for login** → user can't see the URL bar; phishable. Use the system browser (RFC 8252).
- **Wildcard `redirect_uri`** → `https://*.example.com` is an open redirector. Register exact strings.

### Resource server side

- **Skipping `aud`** → confused deputy. Always pin to your RS URI.
- **Caching JWKS forever** → key rotation breaks logins. Cache 10–60 min, refresh on unknown `kid`.
- **Using HS256 for distributed verification** → every RS now needs the secret. Use RS256/ES256.
- **Trusting `scope` without checking it** → you've reinvented "any token works".
- **Logging Authorization headers** → the bearer token ends up in CloudWatch / Datadog. Scrub or use DPoP/mTLS.

### Operational

- Clock skew > 1 minute → tokens look expired or not-yet-valid. NTP everywhere.
- Refresh tokens that never rotate → stolen-token persistence forever.
- Per-environment client IDs leaking into the wrong stage.

---

## Slide 25 — Summary & Where to Next

### What we covered

- Why OAuth exists — killing the password anti-pattern
- OAuth 1.0a → 2.0 → 2.1: a 17-year arc
- Roles, endpoints, tokens (access / refresh / ID)
- Authorization Code Flow + PKCE — the one true path
- Client Credentials & Device Code grants
- Why Implicit & Password grants are gone
- Scopes, audience, Resource Indicators
- OpenID Connect — identity on top
- JWT validation, DPoP, mTLS
- OAuth 2.1 + companion RFCs roadmap
- Common attacks & defences (BCP 240)
- Library & flow selection

### Companion deck — Part 2

**"OAuth for MCP Servers & the Identity Provider Landscape"**

- The Anthropic MCP authorisation profile (2025)
- How remote MCP servers use OAuth 2.1 + Resource Indicators
- The Docker MCP Gateway and its OAuth interceptor
- Commercial providers — Auth0/Okta, Microsoft Entra, AWS Cognito, Stytch, WorkOS, Clerk, Descope
- Free / OSS — Keycloak, Authentik, ZITADEL, Ory Hydra, Authelia, Logto, SuperTokens
- Worked examples and a production checklist

### If you remember three things

1. OAuth = **delegated authorisation**; OIDC = **identity**; everyone needs both.
2. **Authorization Code + PKCE** is the only flow you should reach for when a user is involved.
3. Tokens must be **audience-bound**; ideally **sender-constrained** (DPoP/mTLS).

### One-line takeaway

You don't implement OAuth — you *configure* it. Pick a conformant library, pin every parameter the spec lets you pin, and read BCP 240.

### Further reading

RFC 6749 (core) · RFC 6750 (Bearer) · RFC 7636 (PKCE) · RFC 8414 (AS Metadata) · RFC 8628 (Device) · RFC 8707 (Resource Indicators) · RFC 9068 (JWT AT) · RFC 9449 (DPoP) · RFC 9700 / BCP 240 (Security BCP) · OpenID Connect Core 1.0 · oauth.net · openid.net
