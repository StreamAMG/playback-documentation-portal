# StreamAMG Playback Embed — Authentication & Content Types Guide

This guide explains how to enable **authenticated playback** and how **content access levels** work when using the StreamAMG Playback Embed SDK.

---

# Overview

The StreamAMG embed player reads a user authentication token from **localStorage** using a configurable key.

By default, the player expects the token to be stored under:

```
streamamg_auth_token
```

Your platform is responsible for:

- Storing the token after login
- Removing the token on logout
- Allowing the token to expire naturally

---

# How It Works

1. User logs into your platform  
2. Your platform receives an authentication token (JWT / SSO token)  
3. Your platform stores the token in `localStorage`  
4. The StreamAMG embed reads the token automatically  
5. The token is sent to Playback API  
6. Playback is authorized (or rejected)

---

# Content Types & Authentication

StreamAMG supports three types of video content. Each type has different authentication requirements.

## Free-to-Watch Content

Free-to-watch content is available to all users.

**Requirements:**

- `x-api-key` is required  
- No user login required  
- No authentication token required  

**Summary:**

Anyone can watch the content.

---

## Freemium Content

Freemium content requires the user to be **logged in**, but does **not** require a subscription.

**Requirements:**

- `x-api-key` is required  
- User must be logged in  
- `Authorization: Bearer {token}` is required  

**Summary:**

Users must log in before they can watch the content.

---

## Premium Content

Premium content requires the user to be **logged in and subscribed**.

**Requirements:**

- `x-api-key` is required  
- User must be logged in  
- User must have an active subscription  
- `Authorization: Bearer {token}` is required  

**Summary:**

Users must log in and have the correct entitlement to watch the content.

---



# Example Behaviour

| Content Type | Logged Out | Logged In | Logged In + Subscribed |
|--------------|------------|-----------|-------------------------|
| Free-to-Watch | ✅ Plays | ✅ Plays | ✅ Plays |
| Freemium | ❌ Blocked | ✅ Plays | ✅ Plays |
| Premium | ❌ Blocked | ❌ Blocked | ✅ Plays |

---

# Required Implementation

## Store Token After Login

After your user logs in, store the authentication token:

```javascript
localStorage.setItem("streamamg_auth_token", userAuthToken);
```

Example:

```javascript
localStorage.setItem(
  "streamamg_auth_token",
  "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..."
);
```

---

# Remove Token On Logout

When a user logs out, remove the token:

```javascript
localStorage.removeItem("streamamg_auth_token");
```

This ensures:

- The user is no longer authenticated  
- Protected content cannot be accessed  

---

# Token Expiry

If your authentication tokens expire automatically:

No additional work is required.

When the token expires:

- Playback API will return `401`  
- Player will show login message  

Example message:

```
Please log in to watch this content.
```

Your platform should then:

- Redirect user to login  
- Refresh authentication token  

---

# Custom Storage Key (Optional)

If required, you can configure a custom storage key.

Example embed:

```html
<div
  class="streamamg-embed"
  data-entry-id="ENTRY_ID"
  data-playback-api-key="PLAYBACK_KEY"
  data-bitmovin-license-key="BITMOVIN_KEY"
  data-auth-storage-key="my_custom_auth_key"
></div>
```

Then store token using:

```javascript
localStorage.setItem("my_custom_auth_key", token);
```

---

# Example Full Login Flow

### After Login

```javascript
function onUserLogin(token) {
  localStorage.setItem("streamamg_auth_token", token);
}
```

---

### On Logout

```javascript
function onUserLogout() {
  localStorage.removeItem("streamamg_auth_token");
}
```

---

# Example Embed

```html
<div
  class="streamamg-embed"
  data-entry-id="0_123abc"
  data-playback-api-key="YOUR_API_KEY"
  data-bitmovin-license-key="YOUR_LICENSE"
  data-playback-base-url="https://api.playback.streamamg.com/v1"
  data-auth-storage-key="streamamg_auth_token"
></div>

<script src="https://sdk.playback.streamamg.com/v1/playbackembedplayer.js"></script>
```

---

# Multiple Player Support

Authentication is shared automatically:

```html
<div class="streamamg-embed" data-entry-id="video1"></div>
<div class="streamamg-embed" data-entry-id="video2"></div>
```

Both players will use the same authentication token.

---

# Security Notes

- Tokens are stored in browser localStorage  
- Tokens should be short-lived where possible  
- Tokens should be removed on logout  
- Avoid embedding tokens directly in HTML  

---

# Summary

Your platform must:

- Store token after login  
- Remove token on logout  
- Allow token expiry  
- Embed StreamAMG player  

Once implemented, authenticated playback will work automatically.
