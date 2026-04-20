# StreamAMG Playback — WordPress Embed Guide

This guide explains how to embed a StreamAMG video player into WordPress.

---

# What You Will Receive

You will be provided with embed code similar to:

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

## Step 1 — Open WordPress

1. Log into WordPress admin
2. Navigate to **Pages** or **Posts**
3. Click **Edit**

---

## Step 2 — Add Custom HTML Block

1. Click **+ Add Block**
2. Search for **Custom HTML**
3. Select **Custom HTML** block

---

## Step 3 — Paste Embed Code

Paste your StreamAMG embed code into the block.

---

## Step 4 — Preview

Click **Preview** to confirm player loads correctly.

---

## Step 5 — Publish

Click **Publish** or **Update**

Your video should now appear.

---

# Authentication (Important)

If your content requires login:

- Your platform must store the user token in localStorage
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

```html
data-autoplay="false"
data-muted="false"
```

---

# Troubleshooting

## Player Not Showing

Check:

- The Custom HTML block has been used
- The `<script>` tag has not been removed
- The API key is correct
- The Entry ID is valid

---

## Content Requires Login

- Ensure the token exists in localStorage
- Ensure it is stored under the correct key
- Ensure the token is valid (not expired)

---

# Example Working Embed

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

1. Edit page
2. Add Custom HTML block
3. Paste embed
4. Publish
5. Ensure auth token exists (if required)
