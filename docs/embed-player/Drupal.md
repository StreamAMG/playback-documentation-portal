# StreamAMG Playback — Drupal Embed Guide

This guide explains how to embed a StreamAMG video player into a Drupal page.

This is designed as a simple, step-by-step guide.

---

# What You Will Receive

You will be provided with embed code similar to the following:

```html
<div
  class="streamamg-embed"
  data-entry-id="0_7gxpb2ir"
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

## Step 1 — Log Into Drupal

1. Log into your Drupal admin area
2. Navigate to the page where you want to add the video
3. Click **Edit**

---

## Step 2 — Change Text Format

Drupal may block scripts by default.

Find the **Text Format** dropdown and select:

- Full HTML
- Raw HTML
- Unfiltered HTML

(Your Drupal administrator may need to enable this)

---

## Step 3 — Paste The Embed Code

Paste the full StreamAMG embed code where you want the video to appear.

---

## Step 4 — Save The Page

Click **Save** and view the page.

Your video player should now appear.

---

# Embed Settings Explained

### data-entry-id

The unique ID of the video you want to embed.

---

### data-playback-api-key

Your StreamAMG Playback API key.

---

### data-bitmovin-license-key

Your Bitmovin Player license key.

---

### data-playback-base-url

Playback API endpoint (provided by StreamAMG).

---

### data-auth-storage-key

Defines which localStorage key the player uses to retrieve the user token.

Default:
```
streamamg_auth_token
```

---

### data-autoplay

Controls autoplay behaviour.

| Value | Behaviour |
|------|-----------|
| true | Video plays automatically |
| false | User must click play |

---

### data-muted

Controls audio on load.

| Value | Behaviour |
|------|-----------|
| true | Video starts muted |
| false | Video starts with sound |

---

# Recommended Settings

For best compatibility:

```html
data-autoplay="false"
data-muted="false"
```

---

# If The Player Does Not Appear

Check the following:

## 1. Script Tag Removed

Make sure this is still present after saving:

```html
<script src="https://sdk.playback.qa.streamamg.com/v1/playbackembedplayer.js"></script>
```

If Drupal removes it, your text format is too restrictive.

---

## 2. Text Format Not Full HTML

Switch to:

- Full HTML
- Raw HTML
- Unfiltered HTML

---

## 3. Incorrect Video ID

Check:

```
data-entry-id
```

---

## 4. Invalid Keys

Verify:

- Playback API key
- Bitmovin license key

---

## 5. Browser Console Errors

Open developer tools and check Console for errors.

---

# Authenticated Content

If your video requires login:

- Your platform must store the authentication token in localStorage
- The key must match `data-auth-storage-key`
- The player will automatically use the token if present
- If the token is expired, the player will retry without it for free content

---

# Example Working Embed

```html
<div
  class="streamamg-embed"
  data-entry-id="0_7gxpb2ir"
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

# Summary

To embed a StreamAMG video in Drupal:

1. Edit page
2. Switch to Full HTML
3. Paste embed code
4. Save page
5. Video appears

---

# Need Help

If you experience issues embedding videos in Drupal, contact your StreamAMG support team.
