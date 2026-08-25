---
title: "OAuth 2.0 vs OIDC: Stop Confusing Access With Identity"
date: 2026-08-25
draft: false
tags: ["oauth", "oidc", "security", "backend"]
---

<!-- markdownlint-disable-next-line MD036 -->
_Two specs, one wire protocol, and why you shouldn't confuse access with identity_

---

I've worked with HTTP APIs for a while now, REST mostly, the way most of us start. Recently I was going through API design patterns on roadmap.sh and realized how much I'd been treating "API" as a synonym for "REST endpoint I call with fetch." There's a whole world beyond that, and one rabbit hole I fell into was Access Control and Authentication.

That's where OAuth 2.0 and OIDC showed up, and honestly, once I traced the actual request/response flow instead of just pasting `scope=openid` into a config file, it clicked in a genuinely satisfying way. Two specs, riding on the exact same wire protocol, solving two completely different problems. Almost every login button on the internet is running both of them at once without you noticing.

If you've built "Sign in with Google" or called an API with a bearer token, you've used both. But here's the thing that took me a minute to actually get: **OAuth 2.0 and OIDC are not the same protocol wearing different hats.** Conflating them is how you end up shipping login flows that are subtly broken.

The one-sentence version, tattoo it somewhere:

> **OAuth 2.0 grants access. OIDC proves identity. OIDC is just OAuth 2.0 with an ID card stapled to it.**

Let's break down why that's true, then walk the actual wire protocol, the same way I traced it to make it click for myself.

## OAuth 2.0: "Can This App Touch My Stuff?"

OAuth 2.0 exists to solve one problem: letting App A act on your behalf against Service B, without you ever handing App A your Service B password. That's it. It's **authorization**, full stop. It was never designed to tell anyone who you are.

Four roles, memorize them because everything else is built on this vocabulary:

| Role | What it is |
| --- | --- |
| **Resource Owner** | You. The human who owns the data. |
| **Client** | The app asking for access. |
| **Authorization Server** | Issues tokens after you authenticate + consent. |
| **Resource Server** | Hosts the protected data, checks incoming tokens. |

A typical Authorization Code flow request looks like this:

```
GET /authorize?
  client_id=abc123
  &response_type=code
  &redirect_uri=https://myapp.com/callback
  &scope=photos.read
  &state=xyz789
```

The server bounces the user back with a short-lived code, and your backend swaps it for a real token by POSTing to `/token` with `grant_type=authorization_code`, the code, and your client credentials. You get back an `access_token`, maybe a `refresh_token`, and an expiry.

Note what's *not* in there: anything telling you who logged in. `access_token` is opaque-ish, scoped, and expiring: a keycard, not an ID card. Plain OAuth has no standard concept of "prove this is Alice." People used to fake it by treating "I successfully got a token" as login proof. That's the exact hack OIDC was built to kill.

## OIDC: "No Really, Who Is This?"

OpenID Connect is a thin identity layer bolted onto OAuth 2.0 by the OpenID Foundation (Google, Microsoft, and friends). Same transport, same token endpoint, same redirect dance, but it adds exactly one new artifact that changes everything: the **ID Token**, a signed JWT.

Ask for it by adding `openid` to your scope:

```
GET /authorize?
  client_id=abc123
  &response_type=code
  &redirect_uri=https://myapp.com/callback
  &scope=openid%20profile%20email
  &state=xyz789
  &nonce=n-0S6_WzA2Mj
```

That `nonce` is new too. OAuth doesn't have it. It's there to stop replay attacks against the ID token specifically.

Now the token response has a third citizen alongside `access_token`: an `id_token`. Decode it (it's just a JWT) and you get identity claims, not resource permissions:

```json
{
  "iss": "https://accounts.google.com",
  "sub": "110169484474386276334",
  "aud": "abc123",
  "exp": 1743098765,
  "nonce": "n-0S6_WzA2Mj",
  "email": "user@example.com",
  "name": "Ada Lovelace"
}
```

`sub` is the durable, unique user ID, that's your actual "who is this" answer. `aud` and `nonce` exist so you can verify this token was minted for *you*, right now, not stolen from someone else's flow. And critically: **you validate this yourself, client-side, against the provider's public key**: check the signature, check `iss`, `aud`, `exp`, and `nonce`. No extra round trip needed.

Need more than what's in the token, profile picture, locale, whatever? Spend the access_token you already have on the UserInfo endpoint, the same way you'd call any bearer-token-protected API.

## The Whole Flow, End to End

This is the exact sequence the diagram below maps out. Green steps are pure OAuth, blue steps are what OIDC bolts on:

<div style="text-align: center;">

![OAuth 2.0 and OIDC Connecting Flow Diagram](/images/oauth2-oidc-flow-integration.png)

</div>

1. **Redirect to `/authorize`**: OAuth mechanics, OIDC just adds `scope=openid` + `nonce`.
2. **User authenticates & consents**: OIDC verifies *who*, OAuth verifies *what they're allowing*.
3. **Authorization code response**: pure OAuth, single-use code.
4. **`POST /token`**: pure OAuth, code-for-tokens exchange.
5. **Token response**: OAuth gives you `access_token`; OIDC sneaks in `id_token`.
6. **Validate ID token**: 100% OIDC, doesn't exist in plain OAuth at all.
7-8. **`GET /userinfo`**: OIDC-defined endpoint, called the boring OAuth-bearer-token way.
7. **Call the actual API**: back to plain OAuth, access_token as a bearer credential.

Six steps are OAuth's plumbing. Steps 2, 5 (the id_token part), 6, 7-8 are OIDC riding shotgun. It doesn't reinvent the pipe. It just adds one very specific, cryptographically verifiable envelope for "here's who this is."

## tl;dr

- Only need to hit an API on someone's behalf? **Plain OAuth 2.0.** No identity claims needed.
- Need to know *who's logging in*? **You need OIDC**: `scope=openid`, validate the `id_token`, done.
- Access token = keycard. ID token = ID card. Don't use one where you need the other.

This was the moment Access Control stopped being a black box for me. Turns out "authentication" and "authorization" aren't just interview buzzwords, they're two entirely separate specs that happen to share a token endpoint. If you're coming from the REST/HTTP side of things like I was, tracing this flow end to end is worth the hour.

---

*Further reading: [Auth0's intro to OAuth 2.0](https://auth0.com/intro-to-iam/what-is-oauth-2) for the full grant-type zoo, and [Fortinet's OIDC glossary](https://www.fortinet.com/resources/cyberglossary/oidc) for implementation best practices like PKCE and state parameters.*
