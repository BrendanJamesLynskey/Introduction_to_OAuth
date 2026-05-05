# 🔐 Introduction to OAuth

An interactive Reveal.js presentation covering **OAuth** — the delegated-authorisation framework that runs the modern web. Purpose, history, the OAuth 1.0 → 2.0 → 2.1 arc, every grant type, PKCE, OpenID Connect, JWTs, DPoP/mTLS, security best practice, and how to pick a library and a flow.

## ▶ [Open the Presentation](https://brendanjameslynskey.github.io/Introduction_to_OAuth/)

## 📄 [Markdown Version](presentation.md)

## 🧪 [Companion deck — OAuth for MCP Servers & Providers](https://brendanjameslynskey.github.io/OAuth_for_MCP/)

---

## Contents

| # | Topic | Description |
|---|-------|-------------|
| 01 | Title | Delegate → Authorise → Issue → Verify |
| 02 | Topics | Map of foundations, flows, tokens, modern security |
| 03 | Why OAuth Exists | The password anti-pattern and what OAuth actually delegates |
| 04 | A Brief History | 2006 → 2024 timeline — Twitter, Hammer, IETF, MCP profile |
| 05 | OAuth 1.0a | HMAC signatures, three-legged dance, why it was abandoned |
| 06 | OAuth 2.0 | RFC 6749 reset — bearer tokens, multiple grants, JSON |
| 07 | Roles & Terminology | Resource Owner / Client / AS / RS, endpoints, client types |
| 08 | Tokens | Access vs refresh vs ID — bearer, DPoP, mTLS |
| 09 | Authorization Code Flow | Front- and back-channel, sequence diagram |
| 10 | PKCE | RFC 7636 — code_verifier, S256, mandatory in 2.1 |
| 11 | Deprecated Flows | Implicit, ROPC — and why they're gone |
| 12 | Client Credentials | Machine-to-machine, `private_key_jwt`, mTLS |
| 13 | Device Authorization Grant | RFC 8628 — TVs, CLIs, headless agents |
| 14 | Scopes, Audience, RIs | Permissions, audience binding, RFC 8707 |
| 15 | OpenID Connect | Identity layer, ID tokens, discovery |
| 16 | JWTs in OAuth | Anatomy, validation checklist, classic mistakes |
| 17 | OAuth 2.1 | What's in, what's out, companion RFC roadmap |
| 18 | DPoP & mTLS | Sender-constrained tokens beyond bearer |
| 19 | Common Attacks | Code intercept, CSRF, mix-up, theft, confused deputy |
| 20 | OAuth vs Other | SAML, API keys, WebAuthn, decision rule |
| 21 | Real-World Providers | Google, GitHub, Microsoft Entra, Apple |
| 22 | Libraries | Per-language, client and resource-server side |
| 23 | Choosing a Flow | Decision matrix by client type |
| 24 | Cheat Sheet | Endpoints, params, error codes |
| 25 | Gotchas | Production lessons — client, RS, operational |
| 26 | Summary | Three takeaways and a pointer to Part 2 |

---

## Slide Controls

| Action | Key |
|--------|-----|
| Next / Previous | `→` `←` or swipe |
| Overview | `Esc` |
| Fullscreen | `F` |
| Export to PDF | Append `?print-pdf` to URL, then print |

## Technology

[Reveal.js 4.6](https://revealjs.com) · [highlight.js](https://highlightjs.org) · Playfair Display + DM Sans + JetBrains Mono · inline SVG diagrams.

Single self-contained `index.html` — no build step, no npm, no dependencies to install.

## References

RFC 6749 (OAuth 2.0 Framework) · RFC 6750 (Bearer Token Usage) · RFC 7636 (PKCE) · RFC 8252 (Native Apps) · RFC 8414 (AS Metadata) · RFC 8628 (Device Authorization) · RFC 8693 (Token Exchange) · RFC 8705 (mTLS) · RFC 8707 (Resource Indicators) · RFC 9068 (JWT Profile for Access Tokens) · RFC 9126 (PAR) · RFC 9396 (RAR) · RFC 9449 (DPoP) · RFC 9700 (BCP 240 — Security BCP) · RFC 9728 (Protected Resource Metadata) · OpenID Connect Core 1.0 · oauth.net · openid.net

## See also

- [Cloud_aaS_04_SaaS_Architecture](https://github.com/BrendanJamesLynskey/Cloud_aaS_04_SaaS_Architecture) — B2B identity for multi-tenant SaaS (SAML / OIDC / SCIM federation, Auth-as-a-Service providers, audit logs).
- Series hub: [Cloud `*aaS`](https://github.com/BrendanJamesLynskey/Cloud_aaS_Hub).

## License

Educational use. Code examples provided as-is.
