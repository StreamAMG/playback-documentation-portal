# Resume playback — Live Bitmovin integration

> **Fusion and JWKS clients only** — This tutorial implements **`PUT /v1/entry/{entryId}/resume`**, which is **not** available on **CloudPay-only** Playback sites. CloudPay integrators should use the resume channel StreamAMG provided for their stack (not this HTTP PUT).

Step-by-step guide for **live** and **DVR** streams only. For shared prerequisites, authentication, and the Playback **GET** / **PUT** contract, start at [Resume playback — Bitmovin integration](./Resume-Playback-External-Bitmovin.md). For **on-demand (VoD)**, use [Resume playback — On-demand (VoD) Bitmovin integration](./Resume-Playback-External-Bitmovin-VoD.md).

---

## What you are building (live)

1. **GET** playback → read **`playFrom`** (often a **UNIX timestamp** on the stream timeline, typically **≥ 1,000,000,000**).
2. **Load** Bitmovin, wait until the live source is ready, then **`timeShift(playFrom)`** — not a small **`seek`**.
3. On **pause / end / leave**, **PUT** resume with **`playTime`** = `getCurrentTime()` and **`duration`** = **`0`** (or another **finite** value) when the player reports **`Infinity`**.

Live **`playFrom`** and **`playTime`** must stay on the **same time scale** Bitmovin uses for that stream (usually **`getCurrentTime()`** before each PUT).

---

## Bitmovin APIs you will use (live)

| Task | Bitmovin API | Documentation |
|------|----------------|---------------|
| Create player | `new bitmovin.player.Player(container, config)` | [Getting started — Web Player](https://developer.bitmovin.com/playback/docs/getting-started-with-the-web-player) |
| Load manifest | `player.load(source)` | [`PlayerAPI.load`](https://cdn.bitmovin.com/player/web/8/docs/interfaces/Core.PlayerAPI.html#load) |
| Restore DVR / live position | `player.timeShift(unixSeconds)` | [`PlayerAPI.timeShift`](https://cdn.bitmovin.com/player/web/8/docs/interfaces/Core.PlayerAPI.html#timeShift) |
| Detect live (after load) | `player.isLive()` | [`PlayerAPI.isLive`](https://cdn.bitmovin.com/player/web/8/docs/interfaces/Core.PlayerAPI.html#isLive) |
| Current position for PUT | `player.getCurrentTime()` | [`PlayerAPI.getCurrentTime`](https://cdn.bitmovin.com/player/web/8/docs/interfaces/Core.PlayerAPI.html#getCurrentTime) |
| Duration (often non-finite) | `player.getDuration()` | [`PlayerAPI.getDuration`](https://cdn.bitmovin.com/player/web/8/docs/interfaces/Core.PlayerAPI.html#getDuration) |
| Source ready | `PlayerEvent.SourceLoaded` | [`PlayerEvent.SourceLoaded`](https://cdn.bitmovin.com/player/web/8/docs/enums/Core.PlayerEvent.html#SourceLoaded) |
| Live / DVR concepts | Bitmovin playback docs | [Live streaming](https://developer.bitmovin.com/playback/docs/live-streaming) |

OpenAPI reference for the **PUT** body: [Playback Resume API](../../reference/Playback-Resume-API.yaml).

---

## Step-by-step (live)

### Step 1 — GET playback

Same headers as VoD: **`x-api-key`** and **`Authorization: Bearer`**.

```http
GET /v1/entry/{entryId}
x-api-key: YOUR_PLAYBACK_API_KEY
Authorization: Bearer YOUR_VIEWER_SESSION_TOKEN
```

Example response (resume enabled, DVR/live):

```json
{
  "id": "0_live123",
  "playFrom": 1735689600,
  "media": {
    "hls": "https://example.com/live/12345/master.m3u8"
  }
}
```

- **`playFrom`**: for live/DVR, often **UNIX seconds** (≥ `1000000000`). Do **not** pass this to **`seek(123)`** on a live manifest — use **`timeShift`**.
- If **`playFrom`** is missing, start at the **live edge** (no restore).
- Keep **`entryId`** and the **Bearer token** for PUT.

### Step 2 — Load Bitmovin and restore with `timeShift`

1. Call **`player.load({ hls: … })`** without `startOffset` for UNIX-scale live positions.
2. **Before or right after `load`**, subscribe to **`PlayerEvent.SourceLoaded`**.
3. When the source is ready, if **`playFrom >= 1000000000`**, call **`player.timeShift(playFrom)`**.
4. Add a **timeout fallback** (e.g. 5 seconds) in case **`isLive()`** is still false when **SourceLoaded** fires — some HLS live ladders need this.

Do **not** use **`startOffset`** for UNIX-scale **`playFrom`** on live.

### Step 3 — PUT resume when the viewer stops

```http
PUT /v1/entry/{entryId}/resume
```

Body (live example):

```json
{
  "playTime": 1735689720.5,
  "duration": 0,
  "isPlaying": false
}
```

| Field | Live value |
|-------|------------|
| `playTime` | `player.getCurrentTime()` — usually **UNIX** on the timeline for DVR/live |
| `duration` | **`0`** if `getDuration()` is **`Infinity`**, **`NaN`**, or not ready — **never** let `JSON.stringify` send **`null`** |
| `isPlaying` | `false` on pause, end, or page leave |

---

## Full integration example (live)

Replace **`CONFIG`** placeholders. Requires the Bitmovin Web SDK on `window.bitmovin` ([getting started](https://developer.bitmovin.com/playback/docs/getting-started-with-the-web-player)).

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <title>Live resume — Playback + Bitmovin</title>
  <script src="https://cdn.bitmovin.com/player/web/8/bitmovinplayer.js"></script>
</head>
<body>
  <div id="player-container" style="width:100%;max-width:960px;aspect-ratio:16/9;background:#000;"></div>
  <script>
    var CONFIG = {
      playbackBaseUrl: "https://api.playback.example.com",
      playbackApiKey: "YOUR_PLAYBACK_API_KEY",
      entryId: "0_live123",
      viewerBearerToken: "YOUR_VIEWER_SESSION_TOKEN",
      bitmovinLicenseKey: "YOUR_BITMOVIN_LICENSE_KEY",
    };

    var UNIX_PLAYFROM_THRESHOLD = 1000000000;

    function getPlayTimeSeconds(player) {
      try {
        var t = Number(player.getCurrentTime());
        return Number.isFinite(t) && t >= 0 ? t : 0;
      } catch (e) {
        return 0;
      }
    }

    function getDurationSecondsLive(player, playbackData) {
      try {
        var d = Number(player.getDuration());
        if (Number.isFinite(d) && d > 0) return d;
      } catch (e) {}
      var fromGet = Number(playbackData && playbackData.duration);
      if (Number.isFinite(fromGet) && fromGet > 0) return fromGet;
      return 0;
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
        duration: getDurationSecondsLive(player, playbackData),
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
        duration: getDurationSecondsLive(player, playbackData),
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

    function restoreLivePlayFrom(player, playFrom) {
      if (!playFrom || playFrom < UNIX_PLAYFROM_THRESHOLD) return Promise.resolve();

      return new Promise(function (resolve) {
        var finished = false;
        var PlayerEvent = window.bitmovin.player.PlayerEvent;

        function done() {
          if (finished) return;
          finished = true;
          try {
            player.off(PlayerEvent.SourceLoaded, onSourceLoaded);
          } catch (e) {}
          if (timeoutId) clearTimeout(timeoutId);
          resolve();
        }

        function tryApply() {
          if (finished) return;
          try {
            if (player.isLive && player.isLive()) {
              player.timeShift(playFrom);
              done();
              return;
            }
          } catch (e) {}
        }

        function fallbackApply() {
          if (finished) return;
          try {
            player.timeShift(playFrom);
          } catch (e) {
            console.warn("timeShift failed — viewer may be past DVR window", e);
          }
          done();
        }

        var timeoutId;
        function onSourceLoaded() {
          tryApply();
        }

        player.on(PlayerEvent.SourceLoaded, onSourceLoaded);
        timeoutId = setTimeout(fallbackApply, 5000);
        tryApply();
      });
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

      var restorePromise = restoreLivePlayFrom(player, playFrom);
      await player.load({ hls: playbackData.media.hls });
      await restorePromise;

      attachResumeSaveHooks(player, playbackData);
    }

    main().catch(function (err) {
      console.error(err);
    });
  </script>
</body>
</html>
```

**npm / bundler:** import `Player` and `PlayerEvent` from your Bitmovin package instead of `window.bitmovin`.

---

## Sliding-window live (no full DVR)

If the packager only keeps a short buffer, **`timeShift(playFrom)`** may fail when **`playFrom`** is older than the window. Product choice:

- Fall back to **live edge** (no restore), or
- Clamp to **`getMaxTimeShift()`** per [Bitmovin live docs](https://developer.bitmovin.com/playback/docs/live-streaming).

---

## Live checklist

- [ ] Confirmed with StreamAMG that your site uses **Fusion** or **JWKS** (not CloudPay-only for HTTP PUT resume).
- [ ] **GET** with `x-api-key` + viewer **Bearer**; note **`playFrom`** (often UNIX ≥ `1000000000`).
- [ ] Restore with **`timeShift`** after **SourceLoaded** (with timeout fallback), not **`seek`** to a small offset.
- [ ] **PUT** with **`playTime`** from **`getCurrentTime()`** and **finite** **`duration`** (use **`0`** when duration is **`Infinity`**).
- [ ] Same **`entryId`** and **Bearer token** on GET and PUT.
- [ ] Resume **enabled** on your Playback deployment (Fusion / JWKS).

---

## Live troubleshooting

| Symptom | What to check |
|---------|----------------|
| Resume jumps to wrong place | Used **`seek`** instead of **`timeShift`** for UNIX **`playFrom`**. |
| Never restores | **`SourceLoaded`** not wired before **`load`**, or **`isLive()`** false — rely on **timeout fallback**. |
| **400** with **`duration: null`** | `getDuration()` returned **`Infinity`** — coerce to **`0`** before **`JSON.stringify`**. |
| Restore fails silently | **`playFrom`** outside DVR window — handle fallback to live edge. |
| **400** / **TOKEN_ERROR** | **`entryId`** on PUT ≠ GET (e.g. Kaltura id vs UUID) — use same id as GET. |
| **403** on PUT | **Not Fusion/JWKS** (typically CloudPay-only) — do not use this guide; use your legacy resume integration. |

---

## See also

- [Resume playback — Bitmovin integration](./Resume-Playback-External-Bitmovin.md) (hub)
- [Resume playback — On-demand (VoD) Bitmovin integration](./Resume-Playback-External-Bitmovin-VoD.md)
- [Playback Resume API](../../reference/Playback-Resume-API.yaml)
