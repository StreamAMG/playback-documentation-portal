# Resume playback — On-demand (VoD) Bitmovin integration

Step-by-step guide for **on-demand** video only. For shared prerequisites, authentication, and the Playback **GET** / **PUT** contract, start at [Resume playback — Bitmovin integration](./Resume-Playback-External-Bitmovin.md). For **live / DVR**, use [Resume playback — Live Bitmovin integration](./Resume-Playback-External-Bitmovin-Live.md).

---

## What you are building (VoD)

1. **GET** playback → read **`playFrom`** (seconds from the start of the asset).
2. **Load** Bitmovin with **`startOffset`** or **`seek`** to that position.
3. On **pause / end / leave**, **PUT** resume with **`playTime`** = `getCurrentTime()` and **`duration`** = asset length in seconds.

VoD **`playFrom`** and **`playTime`** are normally **small numbers** (e.g. `0`–`7200`), not UNIX timestamps.

---

## Bitmovin APIs you will use (VoD)

| Task | Bitmovin API | Documentation |
|------|----------------|---------------|
| Create player | `new bitmovin.player.Player(container, config)` | [Getting started — Web Player](https://developer.bitmovin.com/playback/docs/getting-started-with-the-web-player) |
| Load manifest | `player.load(source)` | [`PlayerAPI.load`](https://cdn.bitmovin.com/player/web/8/docs/interfaces/Core.PlayerAPI.html#load) |
| Start at offset on load | `source.startOffset` on source config | [`SourceConfig`](https://cdn.bitmovin.com/player/web/8/docs/interfaces/Core.SourceConfig.html) |
| Seek after load | `player.seek(seconds)` | [`PlayerAPI.seek`](https://cdn.bitmovin.com/player/web/8/docs/interfaces/Core.PlayerAPI.html#seek) |
| Current position for PUT | `player.getCurrentTime()` | [`PlayerAPI.getCurrentTime`](https://cdn.bitmovin.com/player/web/8/docs/interfaces/Core.PlayerAPI.html#getCurrentTime) |
| Length for PUT | `player.getDuration()` | [`PlayerAPI.getDuration`](https://cdn.bitmovin.com/player/web/8/docs/interfaces/Core.PlayerAPI.html#getDuration) |
| Save on pause / end | `player.on(PlayerEvent.Paused, …)` etc. | [`PlayerEvent`](https://cdn.bitmovin.com/player/web/8/docs/enums/Core.PlayerEvent.html) |
| Source ready (optional seek) | `PlayerEvent.SourceLoaded` | [`PlayerEvent.SourceLoaded`](https://cdn.bitmovin.com/player/web/8/docs/enums/Core.PlayerEvent.html#SourceLoaded) |

OpenAPI reference for the **PUT** body: [Playback Resume API](../../reference/Playback-Resume-API.yaml).

---

## Step-by-step (VoD)

### Step 1 — GET playback

Use your **Playback base URL**, **`x-api-key`**, and the viewer **`Authorization: Bearer`** token.

```http
GET /v1/entry/{entryId}
x-api-key: YOUR_PLAYBACK_API_KEY
Authorization: Bearer YOUR_VIEWER_SESSION_TOKEN
```

Example response (resume enabled, viewer has watched before):

```json
{
  "id": "0_abc123",
  "playFrom": 123,
  "duration": "1800",
  "media": {
    "hls": "https://example.com/vod/manifest.m3u8"
  }
}
```

- **`playFrom`**: seconds from the **start** of the VoD asset. If missing, use `0`.
- **`duration`**: may be a **string** in JSON — always coerce with `Number()` before use.
- Keep **`entryId`** and the **Bearer token** for the PUT in later steps.

### Step 2 — Load Bitmovin and restore `playFrom`

**Option A (simplest):** set **`startOffset`** when `playFrom` is a positive offset under `1000000000`:

```javascript
var playFrom = Number(playbackData.playFrom) || 0;
var source = {
  hls: playbackData.media.hls,
  startOffset: playFrom > 0 ? playFrom : undefined,
};
await player.load(source);
```

**Option B:** load without offset, then **`seek`** after the source is ready:

```javascript
await player.load({ hls: playbackData.media.hls });

var playFrom = Number(playbackData.playFrom) || 0;
if (playFrom > 0) {
  player.on(bitmovin.player.PlayerEvent.SourceLoaded, function onLoaded() {
    player.off(bitmovin.player.PlayerEvent.SourceLoaded, onLoaded);
    player.seek(playFrom);
  });
}
```

Use **one** approach per integration; Option A is enough for most VoD HLS assets.

### Step 3 — PUT resume when the viewer stops

Call:

```http
PUT /v1/entry/{entryId}/resume
```

Body (VoD example):

```json
{
  "playTime": 456.2,
  "duration": 1800,
  "isPlaying": false
}
```

| Field | VoD value |
|-------|-----------|
| `playTime` | `player.getCurrentTime()` (seconds from start) |
| `duration` | `player.getDuration()` if finite and &gt; 0, else `Number(playbackData.duration)` |
| `isPlaying` | `false` on pause, end, or page leave |

Wire **Paused**, **PlaybackFinished**, **destroy**, and **pagehide** (see full example below).

---

## Full integration example (VoD)

Copy this into a test page and replace the **`CONFIG`** placeholders. It assumes the Bitmovin Web SDK is loaded globally (`window.bitmovin`), as in [Bitmovin’s getting started guide](https://developer.bitmovin.com/playback/docs/getting-started-with-the-web-player).

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <title>VoD resume — Playback + Bitmovin</title>
  <script src="https://cdn.bitmovin.com/player/web/8/bitmovinplayer.js"></script>
</head>
<body>
  <div id="player-container" style="width:100%;max-width:960px;aspect-ratio:16/9;background:#000;"></div>
  <script>
    var CONFIG = {
      playbackBaseUrl: "https://api.playback.example.com",
      playbackApiKey: "YOUR_PLAYBACK_API_KEY",
      entryId: "0_abc123",
      viewerBearerToken: "YOUR_VIEWER_SESSION_TOKEN",
      bitmovinLicenseKey: "YOUR_BITMOVIN_LICENSE_KEY",
    };

    function getPlayTimeSeconds(player) {
      try {
        var t = Number(player.getCurrentTime());
        return Number.isFinite(t) && t >= 0 ? t : 0;
      } catch (e) {
        return 0;
      }
    }

    function getDurationSeconds(player, playbackData) {
      try {
        var d = Number(player.getDuration());
        if (Number.isFinite(d) && d > 0) return d;
      } catch (e) {}
      var fromGet = Number(playbackData && playbackData.duration);
      return Number.isFinite(fromGet) && fromGet > 0 ? fromGet : 0;
    }

    async function fetchPlayback() {
      var url =
        CONFIG.playbackBaseUrl.replace(/\/$/, "") +
        "/v1/entry/" +
        encodeURIComponent(CONFIG.entryId);
      var res = await fetch(url, {
        headers: {
          Accept: "application/json",
          "x-api-key": CONFIG.playbackApiKey,
          Authorization: "Bearer " + CONFIG.viewerBearerToken,
        },
      });
      if (!res.ok) throw new Error("GET playback failed: " + res.status);
      return res.json();
    }

    async function putResume(player, playbackData, isPlaying) {
      var url =
        CONFIG.playbackBaseUrl.replace(/\/$/, "") +
        "/v1/entry/" +
        encodeURIComponent(CONFIG.entryId) +
        "/resume";
      var body = {
        playTime: getPlayTimeSeconds(player),
        duration: getDurationSeconds(player, playbackData),
        isPlaying: !!isPlaying,
      };
      var res = await fetch(url, {
        method: "PUT",
        headers: {
          Accept: "application/json",
          "Content-Type": "application/json",
          "x-api-key": CONFIG.playbackApiKey,
          Authorization: "Bearer " + CONFIG.viewerBearerToken,
        },
        body: JSON.stringify(body),
      });
      if (!res.ok) {
        console.warn("PUT resume failed", res.status, await res.text().catch(function () { return ""; }));
      }
    }

    function putResumeKeepalive(player, playbackData) {
      var url =
        CONFIG.playbackBaseUrl.replace(/\/$/, "") +
        "/v1/entry/" +
        encodeURIComponent(CONFIG.entryId) +
        "/resume";
      var body = JSON.stringify({
        playTime: getPlayTimeSeconds(player),
        duration: getDurationSeconds(player, playbackData),
        isPlaying: false,
      });
      try {
        fetch(url, {
          method: "PUT",
          keepalive: true,
          headers: {
            Accept: "application/json",
            "Content-Type": "application/json",
            "x-api-key": CONFIG.playbackApiKey,
            Authorization: "Bearer " + CONFIG.viewerBearerToken,
          },
          body: body,
        }).catch(function () {});
      } catch (e) {}
    }

    function attachResumeSaveHooks(player, playbackData) {
      var PlayerEvent = window.bitmovin.player.PlayerEvent;

      function save(isPlaying) {
        putResume(player, playbackData, isPlaying);
      }

      player.on(PlayerEvent.Paused, function () { save(false); });
      player.on(PlayerEvent.PlaybackFinished, function () { save(false); });

      var origDestroy = player.destroy.bind(player);
      player.destroy = function () {
        save(false);
        return origDestroy();
      };

      window.addEventListener("pagehide", function () {
        putResumeKeepalive(player, playbackData);
      });
    }

    async function main() {
      var playbackData = await fetchPlayback();
      var playFrom = Number(playbackData.playFrom) || 0;

      var player = new bitmovin.player.Player(
        document.getElementById("player-container"),
        { key: CONFIG.bitmovinLicenseKey },
      );

      var source = { hls: playbackData.media.hls };
      if (playFrom > 0 && playFrom < 1000000000) {
        source.startOffset = playFrom;
      }

      await player.load(source);
      attachResumeSaveHooks(player, playbackData);
    }

    main().catch(function (err) {
      console.error(err);
    });
  </script>
</body>
</html>
```

**npm / bundler:** import `Player` and `PlayerEvent` from your Bitmovin package instead of `window.bitmovin`; method names are unchanged.

---

## VoD checklist

- [ ] **GET** with `x-api-key` + viewer **Bearer** token; note **`playFrom`** and **`entryId`**.
- [ ] Restore with **`startOffset`** or **`seek`** (seconds from start, usually &lt; `1000000000`).
- [ ] **PUT** on pause / finished / destroy / page hide with finite **`duration`**.
- [ ] Same **`entryId`** and **Bearer token** on GET and PUT.
- [ ] Resume **enabled** on your Playback deployment (Fusion / JWKS).

---

## VoD troubleshooting

| Symptom | What to check |
|---------|----------------|
| No **`playFrom`** on GET | Resume disabled, viewer not authenticated, or no saved position yet. |
| Player always starts at 0 | `playFrom` not applied — verify **`startOffset`** or **`seek`** after **SourceLoaded**. |
| **400** on PUT | Body has **`null`** numbers, wrong **`entryId`**, or entry mismatch — use same id as GET. |
| **403** on PUT | CloudPay-only site — HTTP PUT resume not enabled; see hub doc. |

---

## See also

- [Resume playback — Bitmovin integration](./Resume-Playback-External-Bitmovin.md) (hub)
- [Resume playback — Live Bitmovin integration](./Resume-Playback-External-Bitmovin-Live.md)
- [Playback Resume API](../../reference/Playback-Resume-API.yaml)
