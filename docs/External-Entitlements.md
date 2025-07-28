---
stoplight-id: g05q2o59eh341
---

# External-Entitlements

The Playback Service supports **external entitlement management** via JSON Web Tokens (JWT), giving integrators the flexibility to use their own authentication and authorization systems. There are two integration modes:

* **Full External Entitlement Management (Full JWT Integration)**
* **Partial External Entitlement Management (JWT Enrichment via SSO)**

This guide describes both approaches, required configurations, and implementation best practices.

---

## 1. Full External Entitlement Management (JWKS Integration)

In the **Full** mode, you assume **full responsibility for JWT issuance and key management**, including hosting a JWKS (JSON Web Key Set) endpoint for token verification.

### Overview

* You manage the entire authentication and entitlement process.
* You issue JWTs, host the JWKS endpoint, and rotate/sign your keys.
* The Playback Service consumes your JWT and validates entitlements based on configured claims.

### Steps to Integrate

#### 1. Host a JWKS Endpoint

* **Endpoint must be accessible via HTTPS.**
* **Format:** Standard [JWKS specification](https://datatracker.ietf.org/doc/html/rfc7517).
* **Security:**

  * **Only public keys** are exposed via JWKS.
  * Rotate private keys regularly.
* **Availability:**

  * The endpoint must be **highly available** to prevent service disruption.

**Example JWKS:**

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
    },
    {
      "kty": "RSA",
      "use": "sig",
      "kid": "key2",
      "alg": "RS256",
      "n": "ANOVA9jz2ECKf2xW...zEj1Dl",
      "e": "AQAB"
    }
  ]
}
```

**Field Reference:**

* `kty`: Key type (e.g., RSA)
* `use`: Usage (signature)
* `kid`: Key ID (matches JWT header)
* `alg`: Algorithm (RS256 recommended)
* `n`, `e`: RSA key modulus & exponent

#### 2. JWT Token Requirements

Your JWT **MUST** include the following claims:

| Claim        | Description                             |
| ------------ | --------------------------------------- |
| iss          | Issuer URL (your authentication server) |
| aud          | Audience (`playback.streamamg.com`)     |
| exp          | Expiry timestamp                        |
| sub          | Subject/user ID                         |
| session\_id  | Session identifier                      |
| customer\_id | Customer/user ID                        |
| entry\_id    | Content or asset identifier             |

**Example JWT Payload:**

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

#### 3. Playback Service Configuration

We will configure your Playback Service internally to use your JWKS using the following values, this allows you to be flexible to name your customer, session and entry fields.

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

#### 4. JWKS Caching

* JWKS are **cached** for performance.
* **Cache expiration** should balance security (for key rotation) and low latency.
* The Playback Service will **refresh** the JWKS cache if a JWT references an unknown `kid`.


## 2. Partial External Entitlement Management (JWT Claim Enrichment)

In **Partial** mode, you **embed entitlements directly into your SSO JWT**. The Playback Service inspects specified claims to determine user access rights.

### Overview

* Integrate with your existing SSO solution.
* Add entitlement-related claims (e.g., roles, permissions) to the JWT.
* The Playback Service reads these claims to check user entitlements.

### Steps to Integrate

#### 1. Update Playback Service Configuration

Internally we will extend your playback configuration to specify which JWT claims should be checked for entitlements, this allows you to more flexible with your field names.

```json
"entitlements": {
  "defaultEntryEntitlement": ["*"],
  "jwtClaims": ["roles", "scope", "permissions", "entitlements"]
}
```

* The values in `jwtClaims` are the JWT fields the service will check for entitlement information.

#### 2. JWT Token Structure

Ensure your JWT contains the configured entitlement claims, in this instance you can see "premium" and "subscriber" are the enriched entitlements for your customers.

```json
{
  "iss": "https://your-auth-server/",
  "sub": "user_id",
  "aud": "streamamg",
  "iat": 1729508820,
  "exp": 1729514220,
  "roles": ["premium", "subscriber"]
}
```

#### 3. Playback Service Entitlement Logic

* The service reads the listed JWT claims to **verify entitlements**.
* If entitlements are not present or not valid for the requested content, it **falls back** to traditional entitlement systems (e.g., CloudPay).

---

## FAQs

### What’s the difference between Full and Partial Integration?

* **Full**: You own key management and JWT issuance end-to-end. More control, more responsibility.
* **Partial**: You enrich SSO tokens with entitlements; Playback Service still verifies using SSO and may fallback to internal systems.

### Can I use both Full and Partial at the same time?

No, these are distinct integration paths—choose the one that best fits your architecture.

### How do I choose claims for entitlement checks?

* For Full: Use required claims as specified.
* For Partial: Configure `jwtClaims` to list any fields in your JWT that map to user entitlements.
