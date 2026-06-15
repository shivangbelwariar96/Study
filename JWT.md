This is a substantial topic and the speaker covers a lot of ground — JWT structure, how it differs from session IDs, every claim type, the signing flow, four ways to handle token invalidation, JWE vs JWT, and the JWK exploit. I'll walk through every piece in order with mobile-friendly visuals (380px viewBox) and full narration of every example.

## Part 1 — What JWT is, at the simplest level

A JWT (JSON Web Token) is **a secure way of transmitting information between parties as a JSON object**. That's the original purpose — a transmission format. The "secure" part comes from the token being **digitally signed** (using RSA, HMAC, or another algorithm covered in the speaker's cryptography video), which lets the receiver verify two things: the message hasn't been tampered with, and it really came from who claims to have sent it.

Although JWT was *built* as a transmission format, today it's predominantly used in three places:

- **Authentication** — confirming the user is who they claim to be. "Is this really Shrivang sending the request?"
- **Authorization** — checking what permissions the verified user has. "Shrivang is logged in, but does he have permission to fetch this resource?"
- **Single sign-on (SSO)** — log in once to one application, then access other applications in the same trust circle without re-entering credentials.

## Part 2 — The end-to-end flow

Before getting into the structure, here's how a JWT flows through a real system. The flow involves three parties.

<img width="1472" height="920" alt="image" src="https://github.com/user-attachments/assets/f49f2286-9126-401b-98b2-dc29f451be10" />

One thing worth flagging: in many real architectures the resource server doesn't even need to call the auth server to verify — it just needs the public key (more on this when we get to signatures). The speaker shows the round-trip version because it's the most explicit, but in production the verification often happens locally.

## Part 3 — Why JWT replaced session IDs

Before JWT, the dominant scheme was the **session ID** (often `JSESSIONID` in Java apps). The flow looks superficially similar but works very differently under the hood.

When the user logs in, the server generates a random opaque ID (something like `a7f8b2c1...`) and **stores everything about the session in the database**: which user it belongs to, when it expires, what roles the user has. The server returns just the opaque ID to the client. From then on, every request the client makes carries that ID in a cookie. The server takes the ID, **looks it up in the database**, pulls all the session details, validates the request, then responds.

The single most important thing to notice: **every request hits the database**. The session ID by itself contains no information — it's a pointer. All the data lives server-side.

<img width="1472" height="960" alt="image" src="https://github.com/user-attachments/assets/5d164425-9b06-477e-b7ee-c8d4b8914e6a" />

**JWT inverts this.** All the user information — username, expiry, roles, anything else relevant — is embedded directly in the token. The server can verify the token by checking the signature alone, with no DB lookup. This is what "stateless" means in the JWT context: the server keeps no per-session state.

The cost of that statelessness shows up later, in Part 9 — once you've issued a token you can't easily *take it back*, because the server doesn't have a session row to delete. We'll get there.

## Part 4 — The structure of a JWT

A JWT is three base64-encoded segments joined by periods:

```
<base64(header)>.<base64(payload)>.<base64(signature)>
```

The three parts are: **header**, **payload**, and **signature**. Notice that a real JWT looks like a single long string with two dots in it — there's no whitespace, no JSON brackets visible from the outside. The JSON only appears once you base64-decode each segment.

<img width="1472" height="760" alt="image" src="https://github.com/user-attachments/assets/250d8d3b-ed52-4b8c-b686-2cf0f5400651" />


### A naming wrinkle: JWT vs JWS vs JWE

The speaker flags this carefully because it confuses people in interviews. There are three closely related specs:

- **JWT** (JSON Web Token) — the high-level concept. A token carrying claims.
- **JWS** (JSON Web Signature) — a JWT that's signed but **not** encrypted. The payload is encoded (base64) but readable to anyone who has the token.
- **JWE** (JSON Web Encryption) — a JWT that's encrypted. The payload is unreadable without the decryption key.

In practice, **whenever anyone says "JWT" they almost always mean JWS** — a signed token. Pure JWT without a signature exists in the spec (algorithm = `none`), but it's not used in production and should be rejected (we'll come back to this in Part 12). JWE we'll cover in Part 11.

## Part 5 — The header in detail

The header is a small JSON object describing the token's metadata. Two fields matter:

```json
{
  "typ": "JWT",
  "alg": "RS256"
}
```

- `typ` — almost always `"JWT"`. The speaker mentions never having seen another value.
- `alg` — the signing algorithm. Common values: `RS256` (RSA + SHA-256, asymmetric), `HS256` (HMAC + SHA-256, symmetric), `ES256` (ECDSA). The previous video in the speaker's playlist covers the difference between asymmetric (RSA: separate private and public keys) and symmetric (HMAC: one shared secret).

There can be other fields too — `kid` (key ID) becomes critical in Part 13 when we discuss the JWK exploit.

## Part 6 — The payload in detail: three kinds of claims

The payload is also a JSON object, but it carries the actual data. Each field in it is called a **claim**. The JWT spec classifies claims into three categories.

### Registered claims (reserved names)

These have **standardized short names** with specific meanings defined by the JWT RFC. Every JWT library understands them.

| Name | Meaning |
|---|---|
| `iss` | issuer — who issued the token |
| `sub` | subject — unique user identifier |
| `aud` | audience — who the token is intended for |
| `exp` | expiration time — token is invalid after this |
| `nbf` | not before — token is invalid before this |
| `iat` | issued at — timestamp of issuance |
| `jti` | JWT ID — unique identifier for this token |

`jti` becomes important for revocation (Part 9) — it's the unique handle you'd put on a blacklist.

### Public claims

Custom fields you define, with names that are **commonly understood across multiple parties**. Things like `email`, `country`, `first_name`. They're not in the JWT spec, but they're conventional enough that any consuming application knows what to do with them.

**Hard rule from the speaker**: never put sensitive data (passwords, secrets, credit card numbers) in the payload. The payload is only base64-encoded, not encrypted — anyone who intercepts the token can read it.

### Private claims

Custom fields meant for **internal use** by a specific producer and consumer. The speaker gives the example: the auth server adds a field `AM` that only it understands. When the token gets passed to resource server 1 or 2, they see `AM` but don't know what it means. That's fine — it's not for them.

<img width="1472" height="960" alt="image" src="https://github.com/user-attachments/assets/e5bf11e6-29a9-4b29-b719-d0253338b45e" />


## Part 7 — The signature: how the token becomes tamper-proof

This is the heart of JWT. The signature is what guarantees that no one has modified the header or payload in transit. Here's the construction, step by step:

1. **Base64-encode the header.** `eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9` or similar.
2. **Base64-encode the payload.** `eyJzdWIiOiIxMjM0NSIsImV4cCI6MTcxNDk5OTk5OX0` or similar.
3. **Concatenate** them with a period in between: `<encoded_header>.<encoded_payload>`. This concatenated string is the **message** that will be signed.
4. **Sign the message** with the chosen algorithm and key:
   - For **HMAC** (symmetric): the message and a shared secret key both go into HMAC; out comes the signature.
   - For **RSA** (asymmetric): the message goes through RSA signing with the **private key**; out comes the signature.
5. **Base64-encode the signature.**
6. **Concatenate everything**: `<encoded_header>.<encoded_payload>.<encoded_signature>`. That final string is the JWT.
7. <img width="1472" height="1080" alt="image" src="https://github.com/user-attachments/assets/df248dbe-03df-4c8b-b26b-05f376ede807" />
### Why this works

If an attacker intercepts the JWT and changes anything in the header or payload, the signature no longer matches — because the signature was computed over the **original** message. When the receiver re-runs the verification, the computed signature differs from the one in the token, and verification fails. The attacker would also need to produce a new valid signature, which requires the signing key — which they don't have (or shouldn't).

### Verification differs between HMAC and RSA

- **HMAC** uses the **same key** for signing and verifying. Both the auth server (signer) and the verifier need the shared secret. This is fine if both are under your control, but doesn't work well when the verifier is an untrusted third party.
- **RSA** uses the **private key for signing** and the **public key for verifying**. The public key can be distributed widely — anyone can verify the signature, but only the auth server (which holds the private key) can produce one. This is why most production JWT setups use RSA.

## Part 8 — How the JWT travels: the `Authorization` header

Once issued, the JWT goes back to the client and the client attaches it to every subsequent request. The standard place is the **`Authorization` HTTP header**, prefixed with the word `Bearer`:

```
GET /api/v1/details HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni...
```

Why `Bearer`? The `Authorization` header is a standardized place for credentials, and it supports multiple schemes. The two common ones:

- `Authorization: Basic <base64-of-user:pass>` — you're sending username and password directly.
- `Authorization: Bearer <token>` — you're sending an opaque token. The server treats it as proof of identity for whoever "bears" it.

The server reads the scheme keyword first to know what to do with what follows. `Basic` → decode and check credentials. `Bearer` → validate as a token.

## Part 9 — The big problem: token invalidation

Now the speaker pivots to the hard part. JWT's statelessness is also its biggest weakness.

Here's the scenario: a user logs in today (April 1) and receives a JWT with `exp = April 21`. On April 5, the company decides this user is a fraud and wants to ban them. But the token they hold is still cryptographically valid — its signature checks out, its expiry is still in the future. The user can keep accessing the system for another 16 days.

**How do you revoke a JWT before its natural expiry?** With session IDs this was trivial — delete the row from the database. With JWT, there's nothing to delete. The token is self-contained and the server has no record of having issued it.

The speaker walks through four production approaches, each with tradeoffs.

### Approach 1 — Blacklist (deny list)

Keep a list of revoked token IDs (using `jti`, the registered claim) in a database or cache. On every request, look up the token's `jti` in the blacklist before accepting it.

**Tradeoff**: this brings back the per-request DB/cache lookup that we abandoned session IDs to escape. You haven't fully solved the problem — you've just made the lookup smaller. A cache (Redis) makes it bearable, and the blacklist only grows as long as unrevoked tokens haven't expired yet, so it's bounded.

### Approach 2 — Rotate the signing key

Change the auth server's signing key. Every JWT signed with the old key now fails verification.

**Tradeoff**: this is the nuclear option — it invalidates **every** token, not just the fraud user's. Every legitimate user in the system has to log in again. Useful as an emergency response if you suspect the key was compromised, but not appropriate for routine revocation of one user.

### Approach 3 — Very short-lived tokens

Issue tokens that expire in 5 or 10 minutes instead of days. Even a revoked-but-not-yet-noticed token can only do damage for a few minutes. Pair this with a **refresh token** mechanism: the client gets a short-lived access token (5 min) and a long-lived refresh token (kept securely). When the access token expires, the client trades the refresh token for a new access token. The refresh token itself can be revoked server-side (it's only used at the auth server, not on every request).

**Tradeoff**: doesn't fully solve revocation — the bad user has up to 5 minutes of access after you flag them — but limits the blast radius to an acceptable window.

### Approach 4 — One-time-use tokens

A token is valid only for a single use. After the first request that uses it, the token is marked consumed (typically via its `jti` in a cache) and any subsequent use is rejected.

**Tradeoff**: combines well with short-lived tokens. The implementation requires storing `jti`s in a cache, so you're back to a lookup — but it's a strong guarantee.In practice, **approaches 3 and 4 are often combined**: short-lived tokens that can only be used once. The blacklist (approach 1) becomes the safety net for emergencies.

## Part 10 — Single sign-on (SSO) with JWT

The speaker notes SSO is a common interview question, and JWT is the standard way to implement it. The setup:

You have one auth server and several apps (App 1, App 2, App 3) inside the same trust circle (a company, an ecosystem). The user logs in once to the auth server and gets a JWT. Now when they visit App 1, the browser passes the same JWT — App 1 verifies the signature, reads the payload (which already contains the username, email, roles), and signs them in immediately. No password prompt. Same when they visit App 2 and App 3.

The user typed their password exactly once. Everywhere else, the JWT speaks for them.## Part 11 — JWE: when encoding isn't enough

The speaker raises the second big concern with JWT: **the payload is encoded, not encrypted**. Anyone holding the token can base64-decode the payload and read every claim. For most use cases that's fine — claims are usually things like username and roles, not secrets. But if you genuinely need to put confidential data in the token, plain JWT is the wrong tool.

**JWE (JSON Web Encryption)** is the answer. The structure is similar but instead of just encoding the payload, it **encrypts** the payload with a key. The encoded payload becomes ciphertext that's unreadable without the decryption key.

The same RSA private/public mechanism applies in a flipped direction: for encryption, the producer uses the recipient's **public key** to encrypt; the recipient uses their **private key** to decrypt. (Note the asymmetry from signing — for signatures, the producer's private key signs and any public key verifies. For encryption, anyone's public key encrypts and only the recipient's private key decrypts.)

**Use JWT (JWS) for most cases.** Use JWE only when the payload itself must be confidential — for example, if you're including personal data, tokens for downstream APIs, or anything that mustn't leak even to someone who intercepts the request.

## Part 12 — The `alg: none` pitfall (unsecured JWT)

A pure JWT with no signature is technically valid per the spec — the algorithm field is set to `"none"` and the third segment is empty. Such a token is called an **unsecured JWT**.

The speaker is emphatic: **any production server should reject `alg: none` outright.** It exists in the spec for completeness but has no legitimate use. The reason it's dangerous: an attacker who can intercept a token can strip the signature, change `alg` to `"none"`, modify the payload however they like, and send it back. A naive verifier that trusts the `alg` field would accept this. Many JWT libraries have had `alg: none` vulnerabilities historically — the safe configuration is to maintain an explicit allow-list of accepted algorithms.

This is why "JWT" and "JWS" are used interchangeably in practice. Nobody uses unsigned JWTs.

## Part 13 — The JWK exploit (the trickiest one)

This is the most subtle attack and the speaker spends real time on it. To understand it you need to know what JWK is.

**JWK (JSON Web Key)** is a standard format for representing cryptographic keys as JSON. A public key in JWK form looks something like:

```json
{
  "kty": "RSA",
  "kid": "key-2024-01",
  "n": "<modulus>",
  "e": "<exponent>"
}
```

The `n` and `e` fields together define an RSA public key. The `kid` is a key identifier.

Now here's the dangerous thing: the JWT header **can include a JWK field**, embedding the public key directly inside the token header. The intent was convenience — the verifier could read the public key right out of the token. The exploit:

1. An attacker intercepts a legitimate JWT.
2. Modifies the payload (e.g., changes `role: user` to `role: admin`).
3. **Generates their own RSA key pair.** Replaces the `jwk` in the header with their own public key.
4. Signs the modified token with their own private key.
5. Sends it to the resource server.

A naive verifier that does "extract the public key from the JWT header and verify the signature against it" will succeed — because the attacker's signature *does* match the attacker's public key. The verifier just trusted the wrong key.### The fix: use `kid`, not `jwk`

**Never trust the public key embedded in the JWT itself.** Use the `kid` (key ID) field in the header to *look up* the correct public key from a trusted source — namely, the auth server's published list of public keys at a well-known URL like:

```
https://auth.example.com/.well-known/jwks.json
```

This file is maintained by the auth server and contains every public key it currently uses for signing, each tagged with a `kid`. The verifier:

1. Reads the `kid` from the incoming JWT header.
2. Looks up that `kid` in the trusted JWKS endpoint (cached locally for performance).
3. Uses the public key from there — **not** the one in the JWT header.
4. Verifies the signature.The speaker adds one last caveat: this all assumes the auth server itself is trustworthy and keeps its JWKS endpoint clean. If an attacker can inject their own public key *into* the auth server's JWKS, the exploit still works. So pick a well-known, well-maintained auth provider (Okta, Auth0, AWS Cognito, etc.) — they have the operational maturity to protect their key infrastructure.

## Part 14 — Why JWT wins despite the warts

The speaker closes with a summary of advantages, which I'll consolidate here:

- **Compact** — small enough to fit in an HTTP header. No bulk transmission overhead.
- **Self-contained** — all the data the receiver needs is in the token. No DB lookup on the verification path.
- **Stateless** — the server keeps no session state, so it scales horizontally without session replication.
- **Digitally signed** — tamper-evident and authenticity-verifiable.
- **Built-in expiry** — `exp` claim means even a leaked token has a half-life.
- **Custom claims** — you can pack roles, permissions, or any context your app needs directly into the token, eliminating round trips for authorization decisions.
- **Library ecosystem** — every major language has battle-tested JWT libraries. You don't write the cryptography yourself.

The downsides — invalidation is hard, payload isn't encrypted, `alg: none` and JWK exploits are real attack surfaces — are real but well-understood. Used carefully (short-lived tokens, blacklists for emergencies, JWE for confidential data, `kid`-based verification, explicit algorithm allow-listing), JWT remains the dominant choice for modern authentication.

## The one-paragraph recap

A JWT is a small JSON-based credential carried in the `Authorization: Bearer ...` header, made of three base64-encoded segments — **header** (metadata: `typ` and signing `alg`), **payload** (claims: registered like `iss`/`sub`/`exp`/`jti`, public like `email`, and private custom ones), and **signature** (computed over the concatenated header and payload using HMAC's shared secret or RSA's private key). Because all the user data lives inside the token, the server can verify a request by checking only the signature — no database lookup, no session table, fully **stateless**, which is the property that made JWT supplant session IDs and that makes single sign-on across multiple apps trivial. The price of statelessness is that revoking a token before its `exp` is awkward; the four production strategies are jti-blacklisting, signing-key rotation (nuclear), short-lived tokens with refresh (most common), and one-time-use tokens (often combined with short-lived). Three security pitfalls round out the picture: the payload is **encoded, not encrypted** (use JWE if confidentiality matters), `alg: "none"` unsecured JWTs must always be rejected, and the **JWK exploit** — where an attacker substitutes their own public key into the token header — is defeated by ignoring any embedded `jwk` and instead using the `kid` to look up the public key from the auth server's trusted JWKS endpoint.

If you want, the natural follow-up is OAuth 2.0 — the framework that wraps JWT (specifically as access tokens) into a complete authorization protocol with grant types, scopes, refresh tokens, and the authorization-code flow. That's the spec most "log in with Google" buttons actually implement, and JWT is just one component of it.
