# Resume playback — Bitmovin integration

This guide explains how to integrate **resume playback** with the StreamAMG **Playback API** and **Bitmovin Player**: viewers can leave a video and resume later from the last saved position.

---

## Prerequisites

1. **StreamAMG Playback** is integrated (you have a **Playback API base URL** and **`x-api-key`**).
2. Viewers are **authenticated** for protected content (your app obtains a session / access token accepted by Playback — e.g. Fusion or JWKS flows as agreed with StreamAMG).
3. **Resume is enabled** for your StreamAMG Playback deployment.  
   - This is configured on the StreamAMG side; you do **not** need to edit Playback Configuration yourself. If resume is off, `playFrom` will not be returned and you must **not** call the resume **PUT** endpoint.
4. **HTTP resume updates** via this guide apply when your Playback integration uses **Fusion** or **JWKS** authentication (as set up by StreamAMG). **CloudPay** customers continue to use the **existing** resume mechanism (not this **PUT** endpoint).

---

## What resume does

- Playback stores **one** resume position per viewer and per video **entry**.
- On each **GET**, if a saved position exists, the JSON body can include **`playFrom`** (seconds from the start).
- Your player should **seek** to `playFrom` when it is present and greater than zero.
- When the viewer **pauses**, **finishes**, **leaves the page**, or the **player is destroyed**, you should send a **PUT** so the position is saved for the next visit.

---

## Step 1 — GET playback (same as today)

Request playback for the entry your app already uses (UUID, Kaltura entry id, or other id supplied by your CMS — **use the same id** for GET and for the resume PUT).

```http
GET /v1/entry/{entryId}
x-api-key: YOUR_PLAYBACK_API_KEY
Authorization: Bearer YOUR_VIEWER_SESSION_TOKEN
```

Example:

```bash
curl -sS \
  -H "x-api-key: YOUR_PLAYBACK_API_KEY" \
  -H "Authorization: Bearer YOUR_VIEWER_SESSION_TOKEN" \
  "https://api.playback.example.com/v1/entry/0_abc123"
```

### Response body

If resume is enabled and a position was stored earlier, the response may include:

```json
{
  "id": "0_abc123",
  "name": "Example",
  "playFrom": 123,
  "media": {
    "hls": "https://example.com/manifest.m3u8"
  }
}
```

- **`playFrom`**: number, **seconds**. If omitted, start from `0`.
- Use your **Bitmovin** API to seek after the source is loaded (e.g. `player.seek(playFrom)` or `startOffset` in your setup), following Bitmovin’s docs for your player version.

---

## Step 2 — PUT resume position

When you need to **save** progress, call:

```http
PUT /v1/entry/{entryId}/resume
x-api-key: YOUR_PLAYBACK_API_KEY
Authorization: Bearer YOUR_VIEWER_SESSION_TOKEN
Content-Type: application/json
```

**Important**

- Use the **same `{entryId}`** as in the GET that established the playback session for this asset (must match the entry Playback resolved for that request).
- Use the **same `Authorization` token** you used for GET (**viewer session**), not a different token type.

### JSON body

```json
{
  "playTime": 123.5,
  "duration": 1800,
  "isPlaying": false
}
```

| Field | Meaning |
|-------|--------|
| `playTime` | Current playback position in **seconds** (number). |
| `duration` | Total duration in **seconds** (e.g. from the player or from GET). |
| `isPlaying` | `true` if playback is actively progressing; usually `false` on pause / leave / destroy. |

Example:

```bash
curl -sS -X PUT \
  -H "x-api-key: YOUR_PLAYBACK_API_KEY" \
  -H "Authorization: Bearer YOUR_VIEWER_SESSION_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"playTime":123.5,"duration":1800,"isPlaying":false}' \
  "https://api.playback.example.com/v1/entry/0_abc123/resume"
```

A successful update typically returns **204 No Content** (empty body). Treat any other status per your HTTP client practices.

---

## Step 3 — When to call the PUT (Bitmovin behaviour)

Wire **`PUT`** to **real viewer actions** — not on a tight timer — so you do not spam the API while playback is running.

Recommended hooks:

- **Paused** → send PUT with `isPlaying: false`.
- **Playback finished** → send PUT with `isPlaying: false`.
- **Player destroyed** → send PUT (e.g. `fetch` with **`keepalive`** where the page is unloading).
- **Page hide / tab hidden** → send PUT when the user leaves the tab or navigates away (use **`keepalive`** if the request must outlive the page).

**Throttling:** If you also send periodic updates while playing, throttle (e.g. at most once every few seconds) and skip when the position barely changed.

The **Code examples** section below shows how to read **`playTime`** / **`duration`** from Bitmovin and attach these hooks.

---

## Code examples — Bitmovin Web SDK

Below, **`player`** is your Bitmovin **`Player`** instance (for example from `new bitmovin.player.Player(container, config)` when using the bundled Web SDK).

### Where `playTime` and `duration` come from

| Field | Primary source | Fallback |
|--------|----------------|----------|
| **`playTime`** | `player.getCurrentTime()` | Use `0` if the call throws or the player is already torn down. |
| **`duration`** | `player.getDuration()` | If that is `0`, `NaN`, or not ready yet, use the **`duration`** from your **GET playback** JSON (Playback often returns it as a **string** — coerce with `Number(...)`). |

Example helpers:

```javascript
function getPlayTimeSeconds(player) {
  try {
    if (player && typeof player.getCurrentTime === "function") {
      const t = Number(player.getCurrentTime());
      return Number.isFinite(t) && t >= 0 ? t : 0;
    }
  } catch (e) {
    /* player may be destroyed */
  }
  return 0;
}

function getDurationSeconds(player, playbackPayload) {
  try {
    if (player && typeof player.getDuration === "function") {
      const d = Number(player.getDuration());
      if (Number.isFinite(d) && d > 0) return d;
    }
  } catch (e) {}

  // From GET /v1/entry/... body (may be string)
  if (playbackPayload && playbackPayload.duration != null) {
    const d = Number(playbackPayload.duration);
    if (Number.isFinite(d) && d > 0) return d;
  }
  return 0;
}
```

Use these values when building the PUT body:

```javascript
const playTime = getPlayTimeSeconds(player);
const duration = getDurationSeconds(player, playbackData);
const body = {
  playTime,
  duration,
  isPlaying: false,
};
```

### `PUT` helper (same token and entry as GET)

```javascript
async function sendPlaybackResumePut({
  playbackBaseUrl,
  entryId,
  playbackApiKey,
  viewerBearerToken,
  playTime,
  duration,
  isPlaying,
}) {
  const url =
    String(playbackBaseUrl).replace(/\/$/, "") +
    "/v1/entry/" +
    encodeURIComponent(String(entryId).trim()) +
    "/resume";

  const res = await fetch(url, {
    method: "PUT",
    headers: {
      Accept: "application/json",
      "Content-Type": "application/json",
      "x-api-key": playbackApiKey,
      Authorization: "Bearer " + viewerBearerToken,
    },
    body: JSON.stringify({ playTime, duration, isPlaying }),
  });

  if (!res.ok) {
    console.warn("Playback resume PUT failed", res.status, await res.text().catch(() => ""));
  }
}

/** Convenience: read Bitmovin values and send */
async function saveResumeFromPlayer({
  player,
  playbackData,
  playbackBaseUrl,
  entryId,
  playbackApiKey,
  viewerBearerToken,
  isPlaying,
}) {
  const playTime = getPlayTimeSeconds(player);
  const duration = getDurationSeconds(player, playbackData);

  await sendPlaybackResumePut({
    playbackBaseUrl,
    entryId,
    playbackApiKey,
    viewerBearerToken,
    playTime,
    duration,
    isPlaying: !!isPlaying,
  });
}
```

### Bind Bitmovin events (recommended pattern)

The Bitmovin Web SDK publishes event constants on **`window.bitmovin.player.PlayerEvent`** (same pattern as the StreamAMG embed player). Wire **pause**, **finished**, **destroy**, and optional **page hide**:

```javascript
function attachPlaybackResumeTracking({
  player,
  playbackData,
  playbackBaseUrl,
  entryId,
  playbackApiKey,
  viewerBearerToken,
}) {
  const PlayerEvent = window.bitmovin?.player?.PlayerEvent;
  if (!player || !PlayerEvent) return;

  async function onLeaveOrPause(isPlaying) {
    await saveResumeFromPlayer({
      player,
      playbackData,
      playbackBaseUrl,
      entryId,
      playbackApiKey,
      viewerBearerToken,
      isPlaying,
    });
  }

  if (typeof player.on === "function") {
    if (PlayerEvent.Paused) {
      player.on(PlayerEvent.Paused, function () {
        onLeaveOrPause(false);
      });
    }
    if (PlayerEvent.PlaybackFinished) {
      player.on(PlayerEvent.PlaybackFinished, function () {
        onLeaveOrPause(false);
      });
    }
  }

  const origDestroy =
    typeof player.destroy === "function" ? player.destroy.bind(player) : null;
  if (origDestroy) {
    player.destroy = function () {
      onLeaveOrPause(false);
      return origDestroy();
    };
  }

  function onPageHide() {
    onLeaveOrPause(false);
  }
  window.addEventListener("pagehide", onPageHide);
  document.addEventListener("visibilitychange", function () {
    if (document.visibilityState === "hidden") onPageHide();
  });
}
```

### Page close / navigation: `keepalive` (optional)

Some browsers abort normal `fetch` when the document unloads. For **pagehide** / **beforeunload** you can use `fetch(..., { keepalive: true })` so the request may still complete after navigation:

```javascript
function sendPlaybackResumePutKeepalive({
  playbackBaseUrl,
  entryId,
  playbackApiKey,
  viewerBearerToken,
  playTime,
  duration,
  isPlaying,
}) {
  const url =
    String(playbackBaseUrl).replace(/\/$/, "") +
    "/v1/entry/" +
    encodeURIComponent(String(entryId).trim()) +
    "/resume";

  const body = JSON.stringify({ playTime, duration, isPlaying });

  try {
    fetch(url, {
      method: "PUT",
      keepalive: true,
      headers: {
        Accept: "application/json",
        "Content-Type": "application/json",
        "x-api-key": playbackApiKey,
        Authorization: "Bearer " + viewerBearerToken,
      },
      body,
    }).catch(() => {});
  } catch (e) {}
}

function onPageHideWithKeepalive(player, playbackData, config, viewerBearerToken) {
  const playTime = getPlayTimeSeconds(player);
  const duration = getDurationSeconds(player, playbackData);
  sendPlaybackResumePutKeepalive({
    playbackBaseUrl: config.playbackBaseUrl,
    entryId: config.entryId,
    playbackApiKey: config.playbackApiKey,
    viewerBearerToken,
    playTime,
    duration,
    isPlaying: false,
  });
}
```

Use **`keepalive`** only for small JSON bodies (browser limits apply).

### End-to-end sketch after GET playback

```javascript
// After you have: playbackData from GET JSON, viewerBearerToken, entryId, keys, player instance
const playbackBaseUrl = "https://api.playback.example.com";

attachPlaybackResumeTracking({
  player,
  playbackData,
  playbackBaseUrl,
  entryId,
  playbackApiKey: "YOUR_PLAYBACK_API_KEY",
  viewerBearerToken,
});
```

Adjust **imports** and **`PlayerEvent`** access if you use the **npm** Bitmovin package instead of the global `window.bitmovin` script (the **method names** `getCurrentTime` / `getDuration` / `on` are the same on the player instance).

---

## StreamAMG embed player

If you use the **StreamAMG Playback embed player** (`playbackembedplayer.js`), resume **GET** / **PUT** handling for Bitmovin is implemented in the embed when **resume is enabled** and a viewer token is available. Confirm with your StreamAMG contact that your deployment has resume enabled.

---

## Error handling (recommended)

- **Do not block playback** if a resume **PUT** fails (network, 4xx/5xx). Resume is a convenience; retries are optional.
- **403** on PUT often means this HTTP resume path is **not** enabled for your auth model (e.g. CloudPay-only); use the resume channel StreamAMG provided for your integration.
- **400** may indicate the **entry id in the URL** does not match the entry Playback associated with the authenticated session for that asset — align `{entryId}` with the GET you used for the same playback.

---

## Checklist for integrators

- [ ] GET playback with **`x-api-key`** + **viewer `Authorization`**.
- [ ] Read **`playFrom`** and seek Bitmovin when present.
- [ ] Wire Bitmovin **pause / finished / destroy / page hide** (see **Code examples** above).
- [ ] Same **`entryId`** and **same Bearer token** for GET and PUT.
- [ ] Confirm with StreamAMG that **resume is enabled** for your Playback setup.

---

## Internal reference (StreamAMG staff)

- [Resume playback — internal workflow](./Resume-Playback-Internal.md)
