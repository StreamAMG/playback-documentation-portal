---
internal: true
---

# External Entitlements via JWKS (Full Integration)

The Playback Service can work with **your own authentication and entitlement system** using JSON Web Tokens (JWT).  
This gives you complete control over who can watch content — while StreamAMG validates your tokens securely.

There are two ways to integrate:

- **Full External Entitlement Management** → You handle *everything* (JWT creation + key management).
- **Partial External Entitlement Management** → StreamAMG still handles some verification behind the scenes.

This guide covers the **Full Integration** path.

---

## What “Full External Entitlement” Means

In this mode, **you fully manage the security layer**:
- You issue and sign JWTs.
- You host a JWKS (JSON Web Key Set) endpoint so StreamAMG can verify your tokens.
- You rotate and secure your signing keys.

In short: **StreamAMG will trust your JWTs as the source of truth** for user access.

---

## 1. Set Up a JWKS Endpoint

A **JWKS endpoint** is a public link that lists your public keys, so the Playback Service can verify your JWTs.

**Requirements**
- Must be accessible over **HTTPS**
- Must follow the [JWKS specification (RFC 7517)](https://datatracker.ietf.org/doc/html/rfc7517)
- Should only expose **public keys** (keep private keys secure!)
- Should be **highly available**, since Playback needs it to validate playback tokens

**Example JWKS file:**
```json
{
  "keys": [
    {
      "kty": "RSA",
      "use": "sig",
      "kid": "key1",
      "alg": "RS256",
      "n": "AJOC4Z9b7kTEx1knn...H8Bm0l",
      "e": "AQAB"
    }
  ]
}
```

**Field meanings:**
| Field | What it means |
|-------|----------------|
| `kty` | Key type (usually RSA) |
| `use` | What the key is for (sig = signature) |
| `kid` | Key ID (used to match your JWTs) |
| `alg` | Algorithm used to sign (RS256 recommended) |
| `n`, `e` | Technical parts of the RSA key |

---

## 2. Create JWTs (Your Access Tokens)

Your authentication system must generate JWTs that include certain fields, or *claims*.  
These claims tell Playback who the user is and what content they can access.

**Required Claims:**

| Claim | Description |
|-------|-------------|
| `iss` | Who issued the token (your auth server) |
| `aud` | Who the token is meant for (`playback.streamamg.com`) |
| `exp` | Expiry timestamp (in seconds) |
| `sub` | Unique user ID |
| `session_id` | ID for the current playback session |
| `customer_id` | ID for your customer/user |
| `entry_id` | The specific content or asset ID |

**Example JWT payload:**
```json
{
  "iss": "client.live.com",
  "aud": "playback.streamamg.com",
  "exp": 1700000000,
  "iat": 1699996400,
  "sub": "user12345",
  "session_id": "session67890",
  "customer_id": "user12345",
  "entry_id": "entry98765"
}
```

> **Tip:** Think of this JWT as a “digital ticket” — it proves who the viewer is and what they’re allowed to watch.

---

## 3. Tell StreamAMG Where to Find Your Keys

We’ll configure your Playback Service to use your JWKS details.  
Here’s an example configuration:

```json
{
  "auth": {
    "jwks": {
      "url": "https://your-auth/.well-known/jwks.json",
      "iss": "client.live.com",
      "aud": "playback.streamamg.com",
      "claims": {
        "customer": "sub",
        "session": "jti",
        "entry": "entry_id"
      }
    }
  }
}
```

This tells Playback:
- Where your keys live (`url`)
- What issuer and audience to expect
- Which JWT fields map to StreamAMG’s internal identifiers

---

## 4. Caching & Key Rotation

- Playback caches your JWKS for performance.  
- If your key changes (new `kid`), Playback will automatically fetch the new JWKS.  
- You control how often your keys rotate — we recommend rotating regularly for security.

---

## FAQs

### 🔸 What’s the difference between Full and Partial Integration?

| Mode | Who manages what | Pros | Considerations |
|------|------------------|------|----------------|
| **Full** | You issue tokens & manage keys | Full control, integrate with your own security | Requires more setup & monitoring |
| **Partial** | StreamAMG still verifies some parts | Easier setup | Less flexibility |

---

### 🔸 Can I use both Full and Partial at the same time?
No — they’re separate integration paths. Choose one based on your needs.

---

### 🔸 How do I decide which JWT fields to use for entitlements?
For Full integration, use the required fields listed above.  
For Partial, configure `jwtClaims` to match the data you include in your SSO tokens.

---

## Summary

| You Do | StreamAMG Does |
|---------|----------------|
| Issue and sign JWTs | Validate JWTs against your JWKS |
| Host JWKS endpoint | Cache and refresh JWKS as needed |
| Rotate keys | Use your claims for playback authorisation |

---

**Outcome:**  
Once set up, viewers can start playback only if your JWT says they’re entitled — giving you full, secure control over access.
