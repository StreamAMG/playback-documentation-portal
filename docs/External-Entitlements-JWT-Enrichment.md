---
internal: true
---

# External Entitlements via JWT Enrichment (Partial Integration)

The Playback Service can connect with **your existing login or SSO system** using JSON Web Tokens (JWT).  
This allows you to tell StreamAMG who a viewer is *and* what they’re entitled to watch — directly within your SSO tokens.

There are two types of integrations:

- **Full External Entitlement Management** → You handle all JWT and key management.
- **Partial External Entitlement Management** → You simply include entitlement data in your SSO tokens.

This guide explains the **Partial (JWT Enrichment)** method.

---

## What “Partial External Entitlement” Means

In this mode, you **keep your existing Single Sign-On (SSO)** but add a few extra fields (called *claims*) to your JWT to show what each user can access.

StreamAMG then checks these claims to decide if a viewer is entitled to play a piece of content.

**You manage authentication — we handle entitlement validation.**

---

## 1. Extend Your Playback Service Configuration

We’ll configure your Playback Service to look inside your JWTs for entitlement data.  
You can customise which claim names are used — so it fits your own system.

**Example configuration:**
```json
"entitlements": {
  "defaultEntryEntitlement": ["*"],
  "jwtClaims": ["roles", "scope", "permissions", "entitlements"]
}
```

**How it works:**
- The `jwtClaims` list tells Playback which JWT fields to check for entitlements.  
- Example: If your JWT includes `"roles": ["premium"]`, Playback can allow premium content.  
- The `defaultEntryEntitlement` defines what happens if no specific claim is found.

**Tip:** Use clear claim names like `roles`, `permissions`, or `entitlements` — whatever fits your system best.

---

## 2. Include Entitlement Claims in Your JWT

Your SSO system should issue JWTs that include entitlement fields.  
Here’s an example of what that might look like:

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

**Explanation:**
- The standard JWT fields (`iss`, `sub`, `aud`, `exp`) identify the user and token validity.  
- The `roles` claim lists what the user has access to — in this example, both “premium” and “subscriber” content.

Think of `roles` or `entitlements` as digital “access passes” included in the user’s login token.

---

## 3. How Playback Validates Entitlements

- When a JWT is received, Playback looks for the configured `jwtClaims` fields.  
- If a match is found (e.g., the user’s `role` matches the required content level), playback is allowed.  
- If no valid entitlements are found, Playback **falls back** to StreamAMG’s internal entitlement systems (like CloudPay).

---

## FAQs

### 🔸 What’s the difference between Full and Partial Integration?

| Mode | Who manages what | Pros | Considerations |
|------|------------------|------|----------------|
| **Full** | You issue tokens and manage signing keys | Complete control and flexibility | Requires full key management setup |
| **Partial** | You enrich existing SSO tokens with entitlement data | Easier setup and integration | Limited to claims within your SSO JWT |

---

### 🔸 Can I use both Full and Partial at the same time?
No — these are two separate approaches. Choose one based on your security and integration needs.

---

### 🔸 How do I decide which claims to use for entitlement checks?
For Partial Integration, configure `jwtClaims` to match your JWT field names (e.g., `roles`, `entitlements`, `permissions`).  
This tells Playback exactly where to look for access information.

---

## Summary

| You Do | StreamAMG Does |
|---------|----------------|
| Add entitlement data to your JWTs | Read and validate the claims you specify |
| Manage your SSO and user sessions | Enforce access rules during playback |
| Define which roles/permissions grant access | Fall back to CloudPay if no entitlements are found |

---

**Outcome:**  
Once configured, users can log in through your existing SSO, and Playback automatically knows what they’re allowed to watch — no extra systems needed.
