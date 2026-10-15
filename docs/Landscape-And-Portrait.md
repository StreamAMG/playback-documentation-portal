# Landscape and Portrait

You size the player. `GET /v1/entry/{id}` tells you the shape and returns the poster URL for that shape. The API does not resize the player.

API reference: [Playback API](../reference/Playback-API.yaml) (`GET /v1/entry/{id}`).

---

## 1. Read `portrait` and `coverImg`

`portrait` is always on the entry response.

| `portrait` | Shape | What you do | `coverImg` URLs |
|------------|--------|-------------|-----------------|
| `false`, or the field is missing | 16:9 landscape | Keep the landscape size you already use. | `/width/360`, `/width/720`, `/width/1080` |
| `true` | 9:16 portrait | Set a 9:16 frame so the video fills the player. | `/height/360`, `/height/720`, `/height/1080` |

A portrait video in a 16:9 box is letterboxed. Set 9:16 when you want it to fill the frame.

Use a `coverImg` URL from the response as the poster. Pass that string through unchanged. The keys stay `360`, `720`, and `1080` for both shapes. An entry returns one set.

`thumbnail` is the same image with no size. `thumbnail` and `coverImg` are omitted when the entry has no stored thumbnail. `portrait` is still returned.

The thumbnail host keeps the aspect ratio of the stored still. A portrait URL stays 9:16. A landscape URL stays 16:9. The host may round the requested size to a nearby pixel size.

Landscape entry:

```json
{
  "id": "00000000-0000-4000-8000-000000000001",
  "name": "Match highlights",
  "portrait": false,
  "thumbnail": "https://video-store/test/12345",
  "coverImg": {
    "360": "https://video-store/test/12345/width/360",
    "720": "https://video-store/test/12345/width/720",
    "1080": "https://video-store/test/12345/width/1080"
  },
  "media": {
    "hls": "https://cdn.example/vod/index.m3u8",
    "dash": "https://cdn.example/vod/index.mpd"
  }
}
```

Portrait entry:

```json
{
  "id": "00000000-0000-4000-8000-000000000002",
  "name": "Portrait clip",
  "portrait": true,
  "thumbnail": "https://video-store/test/12345",
  "coverImg": {
    "360": "https://video-store/test/12345/height/360",
    "720": "https://video-store/test/12345/height/720",
    "1080": "https://video-store/test/12345/height/1080"
  },
  "media": {
    "hls": "https://cdn.example/vod/index.m3u8",
    "dash": "https://cdn.example/vod/index.mpd"
  }
}
```

`spriteUrl` is separate. It is returned for CloudMatrix on-demand only. See [Sprite previews](./Sprite-Previews.md).

---

## 2. Add the two frames to your page

Put the player element and both frames in the page. Landscape is 16:9. Portrait is 9:16.

```html
<style>
  #player.player-landscape {
    width: 100%;
    max-width: 960px;
    aspect-ratio: 16 / 9;
  }

  #player.player-portrait {
    width: min(100%, calc(90vh * 9 / 16));
    aspect-ratio: 9 / 16;
    max-height: 90vh;
    margin-inline: auto;
  }
</style>

<div id="player"></div>
```

The portrait width stays inside the parent and keeps the player from growing taller than the viewport. Change `960px` or `90vh` to match your layout.

---

## 3. Apply the frame, then load the player

Do this after `GET /v1/entry/{id}` returns, and again when the viewer picks a different entry.

```javascript
const entry = await fetch(`https://api.playback.streamamg.com/v1/entry/${entryId}`, {
  headers: { 'x-api-key': apiKey }
}).then((response) => response.json());

const playerElement = document.getElementById('player');
const isPortrait = entry.portrait === true;

playerElement.classList.toggle('player-portrait', isPortrait);
playerElement.classList.toggle('player-landscape', !isPortrait);

const poster = entry.coverImg?.['720'] ?? entry.thumbnail;

player.load({
  title: entry.name,
  hls: entry.media.hls,
  dash: entry.media.dash,
  poster
});
```

`entry.portrait === true` is portrait. Every other value, including a missing field, is landscape.

Pick `coverImg['360']`, `coverImg['720']`, or `coverImg['1080']` for the poster size you want. `720` is a good default.

If the response includes `spriteUrl`, pass it as `thumbnailTrack`. See [Sprite previews](./Sprite-Previews.md).

---

## Playback SDK

The Playback SDK does not change the player size. Use the same CSS from step 2. After `Playback.play` resolves, read the flag and set the class.

```javascript
await Playback.play({
  container: 'player',
  entryId,
  token
});

const details = Playback.getVideoDetails('player');
const playerElement = document.getElementById('player');
const isPortrait = details.portrait === true;

playerElement.classList.toggle('player-portrait', isPortrait);
playerElement.classList.toggle('player-landscape', !isPortrait);
```

Run that again after playlist `next`, `prev`, and `goTo`. `details.portrait` is `true` only for a portrait entry.

New projects should call `GET /v1/entry/{id}` directly. See [SDK General](./SDK/General.md).

---

## Preview

`GET /v1/preview/{id}` returns `portrait` as well. Preview `coverImg` URLs stay `/width/360`, `/width/720`, and `/width/1080` for both orientations. Use `GET /v1/entry/{id}` when you need the portrait `/height/` posters.

---

## Checklist

- Read `portrait` on `GET /v1/entry/{id}` before you choose the player size.
- Keep your landscape size when `portrait` is `false` or missing.
- Add the `player-portrait` class when `portrait` is `true`.
- Pass `coverImg['720']` (or `360` or `1080`) to the player as `poster`.
- Confirm a landscape poster URL contains `/width/` and a portrait poster URL contains `/height/`.
- On the embed, set a height or `aspect-ratio` only when you want to replace the default frame.
- Leave `spriteUrl` off live and Media Platform playback. It is CloudMatrix on-demand only.
