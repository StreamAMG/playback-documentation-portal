# Sprite previews — Bitmovin integration

Seek-bar preview images for **CloudMatrix on-demand** video. `GET /v1/entry/{id}` returns `spriteUrl` when a sprite sheet is available. Pass that URL to Bitmovin as `thumbnailTrack`.

The Playback SDK and the StreamAMG embed player already do this. Use this page when you create a Bitmovin player yourself.

API reference: [Playback API](../reference/Playback-API.yaml) (`GET /v1/entry/{id}`).

---

## When `spriteUrl` is returned

| Video | `spriteUrl` on `GET /v1/entry/{id}` |
|-------|-------------------------------------|
| CloudMatrix on-demand, after transcode | WebVTT URL |
| CloudMatrix live | Omitted |
| Media Platform on-demand | Omitted |
| Media Platform live | Omitted |

The value is the URL of a WebVTT file. Bitmovin reads the cues and shows the matching frame while the viewer scrubs the seek bar. If the field is missing, load the source without `thumbnailTrack`. Playback still succeeds.

---

## 1. Pass the URL when you load the source

```javascript
const source = {
  title: entry.name,
  hls: entry.media.hls,
  dash: entry.media.dash
};

if (entry.spriteUrl) {
  source.thumbnailTrack = { url: entry.spriteUrl };
}

player.load(source);
```

Use the same property on `player.load()` or on the source object you pass when the player is created.

---

## 2. Use a player build that includes thumbnails

The full Bitmovin script already includes thumbnail support. No extra module is required if you load either of these:

- `https://cdn.bitmovin.com/player/web/8/bitmovinplayer.js`
- `https://cdn.jsdelivr.net/npm/bitmovin-player@8/bitmovinplayer.min.js`

A modular player does not include thumbnails until you add the module. Add it once, before you create the player. Add the WebVTT subtitle module first, because the sprite file is a WebVTT track.

```javascript
import { Player } from 'bitmovin-player/modules/bitmovinplayer-core';
import SubtitlesVTTModule from 'bitmovin-player/modules/bitmovinplayer-subtitles-vtt';
import ThumbnailModule from 'bitmovin-player/modules/bitmovinplayer-thumbnail';

Player.addModule(SubtitlesVTTModule);
Player.addModule(ThumbnailModule);
```

If the Thumbnail module is missing, the seek label still shows the time. The preview image does not appear.

---

## 3. Keep the Bitmovin seek-bar thumbnail in the UI

The default Bitmovin UI draws the preview. A custom UI needs the seek-bar thumbnail component. A UI that only draws the time will not show the image.

You can also read the frame yourself with `player.getThumbnail(time)` and draw that image in your own scrub control.

UI class reference: [Video Player Configuration](./Video-Players.md).

---

## 4. Set the preview width

Bitmovin’s own thumbnail width is 6em. The image height follows the frame size in the WebVTT file, so you only set the width. 12em is a comfortable preview size. A rule on `.bmpui-seekbar-thumbnail` alone loses to Bitmovin’s stylesheet.

```css
.bmpui-ui-seekbar-label .bmpui-seekbar-thumbnail {
  width: 12em !important;
}
```

---

## Check

- Call `/v1/entry/{id}` for a CloudMatrix on-demand video and confirm `spriteUrl` is a `.vtt` URL.
- Load the player with `thumbnailTrack: { url: spriteUrl }`.
- On a modular build, confirm `Player.addModule(ThumbnailModule)` runs before `new Player()`.
- Scrub the seek bar and confirm the preview image appears, not only the time.
- Confirm a live or Media Platform entry still plays when `spriteUrl` is absent.
