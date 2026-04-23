---
internal: true
---
# StreamAMG Playback Embed — Flow Overview

This document provides a high-level overview of how the StreamAMG Playback Embed Player works, from initial page load through to video playback or error handling.

---

## Flow Diagram

![StreamAMG Playback Flow](../../assets//images/embed/embed-flow-diagram.png)

---

## Overview

The StreamAMG Playback Embed Player is a lightweight, drop-in JavaScript solution that allows clients to embed video playback into any CMS or website (e.g. Drupal, WordPress, Contentful).

The player:

- Dynamically loads the Bitmovin Player SDK
- Calls the StreamAMG Playback API
- Handles authentication (including expired token fallback)
- Supports configurable token storage keys
- Supports optional Bitmovin Analytics integration
- Displays the video or a user-friendly error message

---

## Example Embed

```html
<div
  class="streamamg-embed"
  data-entry-id="0c8ae54f-ac65-4cbd-83de-ee83926e0946"
  data-playback-api-key="YOUR_PLAYBACK_API_KEY"
  data-bitmovin-license-key="YOUR_BITMOVIN_LICENSE_KEY"
  data-bitmovin-analytics-key="OPTIONAL_ANALYTICS_KEY"
  data-playback-base-url="https://api.playback.streamamg.com/v1"
  data-auth-storage-key="streamamg_auth_token"
  data-autoplay="false"
  data-muted="false"
></div>

<script src="https://sdk.playback.streamamg.com/v1/playbackembedplayer.js"></script>
```

---

## Key Steps

### 1. Embed Initialisation
- The client adds a simple HTML embed snippet to their page
- The SDK locates all `.streamamg-embed` elements or uses a global config

---

### 2. Configuration Handling
The SDK reads configuration from:
- HTML `data-*` attributes **or**
- `window.StreamAMGEmbedConfig`

Required fields:
- `entryId`
- `playbackApiKey`
- `bitmovinLicenseKey`
- `bitmovinAnalyticsKey`
- `playbackBaseUrl`
- `authStorageKey`

Optional fields include:
- `autoplay`
- `muted`

---

### 3. Player Frame Setup
- The SDK enforces a fixed 16:9 aspect ratio
- Ensures consistent sizing across playback and error states
- No client CSS required

---

### 4. Authentication Handling

The SDK resolves a token in the following order:

1. `data-auth-token` (if provided)
2. `localStorage` using `data-auth-storage-key`
3. Default storage key: `streamamg_auth_token`

---

### 5. Playback API Request

The SDK calls:

`GET /v1/entry/{entryId}`

Headers:
- `x-api-key` (always required)
- `Authorization: Bearer {token}` (if available)

If a token is present and the first request returns:

- `401`
- `reason = NOT_AUTHENTICATED`

the SDK retries the same request **without** the `Authorization` header.

---

### 6. Response Handling

#### Success
- Extract HLS stream URL (`data.media.hls`)
- Build poster image from available sources
- Initialise Bitmovin player
- Schedule adverts if present in the playback payload

#### Error Handling
Errors are mapped to user-friendly messages:

| Status | Behaviour |
|--------|----------|
| 401 | Login required |
| 401 (`NO_ENTITLEMENT`) | Subscription required |
| 401 (`NO_SUBSCRIPTION`) | Uses API message or fallback |
| 403 | Access denied |
| 404 | Uses API message directly |

---

### 7. Player Initialisation

- Bitmovin player is dynamically loaded
- Subtitle safe-area styles are injected automatically
- Player is initialised with:
  - Bitmovin license key (**required**)
  - Optional analytics configuration
  - Title / description / poster
  - HLS stream
  - Advertising support
  - Inline playback support

---

### 8. Analytics (Optional)

If `data-bitmovin-analytics-key` is provided:

- Analytics is enabled automatically
- A viewer ID is resolved from:
  - Playback metadata OR
  - localStorage (guest fallback)
- No configuration required by the client beyond providing the key

If not provided:
- Player runs normally without analytics

---

### 9. Mute & Autoplay Logic

- If `autoplay = true`, video may be forced to start muted
- The SDK explicitly re-applies mute/unmute state after load
- Prevents Bitmovin stored preferences overriding requested state

---

### 10. Playback Behaviour

| Scenario | Outcome |
|----------|--------|
| Autoplay enabled | Attempts `player.play()` with retries |
| Autoplay blocked | Logs debug / warning only |
| Autoplay disabled | Waits for user interaction |

---

### 11. Advert Scheduling

If advert configuration is present:

- Merge global + entry adverts
- Normalize advert types (VAST / VMAP)
- Filter invalid URLs
- Schedule ads via Bitmovin

---

## Design Principles

The SDK is designed to be:

- Plug-and-play (no client CSS required)
- CMS agnostic
- Consistent UI
- Resilient to expired tokens
- Backwards compatible
- Optional analytics support

---

## Summary

The StreamAMG Playback Embed Player:

1. Loads on any webpage
2. Resolves configuration and authentication
3. Calls the Playback API
4. Retries anonymously when needed
5. Initialises Bitmovin (+ analytics if enabled)
6. Plays video or shows an error
