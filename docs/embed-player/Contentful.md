# StreamAMG Playback — Contentful Embed Guide

This guide explains how to embed a StreamAMG video player in Contentful.

---

# What You Will Receive

You will receive embed code like:

```html
<div
  class="streamamg-embed"
  data-entry-id="ENTRY_ID"
  data-playback-api-key="YOUR_PLAYBACK_API_KEY"
  data-bitmovin-license-key="YOUR_BITMOVIN_LICENSE_KEY"
  data-playback-base-url="https://api.playback.qa.streamamg.com/v1"
  data-auth-storage-key="streamamg_auth_token"
  data-autoplay="false"
  data-muted="true"
></div>

<script src="https://sdk.playback.qa.streamamg.com/v1/playbackembedplayer.js"></script>
```

---

# Step-by-Step Guide

## Step 1 — Open Contentful

1. Log into Contentful
2. Open your content entry
3. Locate a Rich Text or HTML field

---

## Step 2 — Insert Embed

Paste the embed code into the field.

> If you are using Rich Text, ensure you insert this into an **embedded HTML / code block** (not plain text), otherwise the script tag may be stripped.

---

## Step 3 — Save Entry

Click **Save**

---

## Step 4 — Publish Entry

Click **Publish**

---

# Authentication (Important)

If your content requires login:

- Your platform must store the user token in localStorage under the configured key
- Default key: `streamamg_auth_token`
- You can override using:
  ```html
  data-auth-storage-key="your_custom_key"
  ```

The player will:
- Use the token if present
- Retry without the token if it is expired (for free content)

---

# Recommended Settings

Disable autoplay for best UX:

```html
data-autoplay="false"
data-muted="false"
```

---

# Troubleshooting

## Video Not Showing

Check:

- The embed is added in an HTML-capable field
- The `<script>` tag has not been removed by Contentful sanitisation
- The API key is correct
- The Entry ID is valid
- The playback base URL is correct

---

## Content Requires Login

- Ensure the token exists in localStorage
- Ensure it is stored under the correct key
- Ensure the token is valid (not expired)

---

# Example

```html
<div
  class="streamamg-embed"
  data-entry-id="0_7gxpb2ir"
  data-playback-api-key="YOUR_KEY"
  data-bitmovin-license-key="YOUR_LICENSE"
  data-auth-storage-key="streamamg_auth_token"
  data-autoplay="false"
  data-muted="true"
></div>

<script src="https://sdk.playback.qa.streamamg.com/v1/playbackembedplayer.js"></script>
```

---

# Summary

1. Edit entry
2. Paste embed
3. Publish
4. Ensure auth token exists (if required)
