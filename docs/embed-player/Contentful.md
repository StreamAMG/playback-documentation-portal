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
  data-autoplay="false"
  data-muted="true"
></div>

<script src="https://sdk.playback.qa.streamamg.com/v1/playbackembedplayer.js"></script>
```

---

# Step‑by‑Step Guide

## Step 1 — Open Contentful

1. Log into Contentful
2. Open your content entry
3. Locate Rich Text or HTML field

---

## Step 2 — Insert Embed

Paste embed code into field

---

## Step 3 — Save Entry

Click **Save**

---

## Step 4 — Publish Entry

Click **Publish**

---

# Recommended Settings

Disable autoplay for best UX

```html
data-autoplay="false"
data-muted="false"
```

---

# Troubleshooting

## Video Not Showing

Check:

- The custom HTML block has been used.
- The script tag hasn't been removed.
- The api key in the embed is correct.
- The entry ID is correct/valid.

---

# Example

```html
<div
  class="streamamg-embed"
  data-entry-id="0_7gxpb2ir"
  data-playback-api-key="YOUR_KEY"
  data-bitmovin-license-key="YOUR_LICENSE"
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
