# Resume playback — Bitmovin integration

This guide explains how to integrate **resume playback** with the StreamAMG **Playback API** and **Bitmovin Player**: viewers can leave a video and resume later from the last saved position. It covers **on-demand (VoD)** and **live** (including DVR) — same endpoints, with the player semantics called out so integrators can self-serve without guessing **`playFrom`** / **`playTime`** behaviour.

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
- On each **GET**, if a saved position exists, the JSON body can include **`playFrom`** (see [On-demand (VoD) versus live](#on-demand-vod-versus-live) — the meaning of this number depends on content type).
- Your player must **restore** that position using the correct Bitmovin API (**seek**, **startOffset**, or **timeShift** — not the same for every asset).
- When the viewer **pauses**, **finishes**, **leaves the page**, or the **player is destroyed**, you should send a **PUT** so the position is saved for the next visit.

---

## On-demand (VoD) versus live

The **HTTP contract is identical** for VoD and live: same **`GET /v1/entry/{entryId}`**, same **`PUT /v1/entry/{entryId}/resume`**, same JSON fields, same headers (`x-api-key`, `Authorization`). What changes is how Bitmovin exposes **time** and how you **apply** a stored position after GET.

### How to tell VoD from live in your app

Use your own content model (CMS type, `isLive` from Playback, manifest URL, etc.) **and** Bitmovin’s **`player.isLive()`** after the source is loaded. Do not rely on `playFrom` alone to infer VoD vs live.

### `playFrom` in the GET response

| Content | Typical `playFrom` scale | Meaning |
|--------|---------------------------|--------|
| **VoD** | Usually **&lt; 1,000,000,000** (e.g. `0` … a few hours in seconds) | **Offset in seconds** from the start of the asset. |
| **Live** (incl. DVR / sliding window) | Often **≥ 1,000,000,000** | **Absolute UNIX time** on the media timeline (seconds since epoch), as used by Bitmovin **`timeShift`** for that stream. |

Some VoD assets may still use large timestamps depending on encoder or packager; the StreamAMG **Playback SDK** uses the **1e9 boundary** as a practical heuristic: values **below** `1000000000` are applied as **start offset** at load time; values **from** `1000000000` are restored after load with **VoD → `seek`**, **live → `timeShift`**, once the source is ready (see Bitmovin **SourceLoaded** / equivalent in your player version).

**Integrator rule:** whatever **`playFrom`** value the Playback **GET** returned for this entry, treat **`PUT` `playTime`** as the same kind of value Bitmovin reports at save time — normally **`player.getCurrentTime()`** right before the PUT. That keeps VoD offsets and live UNIX positions consistent across sessions.

### Restoring `playFrom` in Bitmovin (after GET)

| Situation | Recommended approach |
|-----------|----------------------|
| **VoD**, small `playFrom` (typical offset) | Prefer **`startOffset`** on the source config **or** **`player.seek(playFrom)`** after **`SourceLoaded`** (per Bitmovin docs for your SDK version). |
| **Live**, UNIX-scale `playFrom` | Use **`player.timeShift(playFrom)`** (not `seek` to a small number). Subscribe to **`SourceLoaded`** (or your player’s equivalent) before calling **`load`**, because **`isLive()`** may be unreliable until the engine has prepared the live source; a short **timeout fallback** (e.g. a few seconds) avoids never restoring if the event ordering differs. |
| **Sliding-window live** (no full DVR) | The viewer may only be able to resume within the still-available window; if **`timeShift`** fails, fall back to **live edge** per your product rules. |

If you use the **StreamAMG Playback SDK** or **embed player**, this VoD/live **`playFrom`** handling is implemented for you when resume is enabled — custom Bitmovin integrations must implement the equivalent behaviour.

### PUT body: `playTime`, `duration`, and `isPlaying`

| Field | VoD | Live |
|-------|-----|------|
| **`playTime`** | **`player.getCurrentTime()`** — offset in seconds from asset start (≥ 0). | Usually **`player.getCurrentTime()`** — often **UNIX seconds** on the timeline for DVR/live; must be a **finite** number. |
| **`duration`** | Total length in seconds from **`player.getDuration()`** or from the **GET** playback JSON (`duration` may be a string — coerce with `Number`). | Bitmovin often reports **`Infinity`** for live duration. **`Infinity` serializes to JSON `null`**, which the API **rejects** (400). **Always send a finite number:** use **`0`** if the player has no finite duration, or fall back to a finite **`duration`** from **GET** if your packager provides one. Never omit required fields. |
| **`isPlaying`** | `false` on pause / end / leave; `true` only while actively playing if you send mid-play updates. | Same as VoD. |

The API validates **`playTime`**, **`duration`**, and **`isPlaying`** as required, numeric (where applicable), and **non-negative** — plan your client defaults accordingly.

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

- **`playFrom`**: number. If omitted, start from `0`. For **VoD** this is usually **seconds from the start** of the asset; for **live / DVR** it may be a **UNIX timestamp** on the stream timeline — see [On-demand (VoD) versus live](#on-demand-vod-versus-live).
- Use your **Bitmovin** API to restore the position after the source is ready (**`seek`** / **`startOffset`** for typical VoD; **`timeShift`** for UNIX-scale live positions), following Bitmovin’s docs for your player version.

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

**VoD (typical offset + finite duration):**

```json
{
  "playTime": 123.5,
  "duration": 1800,
  "isPlaying": false
}
```

**Live / DVR (UNIX-scale `playTime`; duration often unknown from the player — use `0` rather than `null`):**

```json
{
  "playTime": 1735689600.12,
  "duration": 0,
  "isPlaying": false
}
```

| Field | Meaning |
|-------|--------|
| `playTime` | Current playback position: for **VoD**, usually **seconds from start**; for **live / DVR**, often **UNIX seconds** on the media timeline — use the same scale Bitmovin **`getCurrentTime()`** uses for that entry (must be a **finite** number ≥ 0). |
| `duration` | Total duration in **seconds** when known (**VoD**). For **live**, use a finite value from GET or **`0`** if the player only reports **`Infinity`** — see [On-demand (VoD) versus live](#on-demand-vod-versus-live). |
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
| **`duration`** | `player.getDuration()` | If that is `0`, **`NaN`**, **`Infinity`**, or not ready yet, use the **`duration`** from your **GET playback** JSON (Playback often returns it as a **string** — coerce with `Number(...)`). **`Infinity` must never reach `JSON.stringify`** for this field — the API expects a finite number (use **`0`** as last resort, especially on **live**). |

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
      // Bitmovin live streams often report Infinity — Number.isFinite rejects it (JSON would otherwise emit null).
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
- **400** may mean:
  - **`entryId`** in the URL does not match the entry Playback associated with the authenticated session (**TOKEN_ERROR** / entry mismatch) — use the **same** id as on GET for that session (for Media Platform entries, the canonical id is often the **Kaltura entry id**, not only the internal UUID).
  - **Invalid body** — missing fields, non-numbers, or **`null`** where a number is required. Common client bug: **`duration: null`** because **`JSON.stringify({ duration: Infinity })`** — always coerce to a **finite** number (see [On-demand (VoD) versus live](#on-demand-vod-versus-live)).

---

## Checklist for integrators

- [ ] GET playback with **`x-api-key`** + **viewer `Authorization`**.
- [ ] Confirm **resume is enabled** for your Playback setup (otherwise **`playFrom`** will not appear and you must not call PUT).
- [ ] **VoD:** restore **`playFrom`** with **`seek` / `startOffset`** as appropriate; **live (UNIX `playFrom`):** restore with **`timeShift`** after the source is ready, not a blind **`seek`**.
- [ ] **PUT:** send **`playTime`** from **`getCurrentTime()`** (same semantics as **`playFrom`** for that asset); send **finite** **`duration`** (use **`0`** on live when the player reports **`Infinity`**).
- [ ] Wire Bitmovin **pause / finished / destroy / page hide** (see **Code examples** above); use **`keepalive`** where the page may unload.
- [ ] Same **`entryId`** and **same Bearer token** for GET and PUT.

---
