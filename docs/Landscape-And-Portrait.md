# Landscape and Portrait

`GET /v1/entry/{id}` tells you whether a still is landscape or portrait, and returns the poster URL for that shape. Use the URL in the response. Do not build a `/width/` URL yourself.

The Playback SDK and the StreamAMG embed player already read these fields. Use this page when you create a player yourself.

API reference: [Playback API](../reference/Playback-API.yaml) (`GET /v1/entry/{id}`).

---

## How to tell them apart

| `portrait` | Shape | `coverImg` URLs |
|------------|--------|-----------------|
| `false` | 16:9 landscape | `/width/360`, `/width/720`, `/width/1080` |
| `true` | 9:16 portrait | `/height/360`, `/height/720`, `/height/1080` |

`portrait` is always returned. `false` includes entries that have no orientation flag.

`thumbnail` is the unsized image. `coverImg` is the same image at three sizes. The keys stay `360`, `720`, and `1080`. An entry returns one set. A landscape response does not include `/height/` URLs, and a portrait response does not include `/width/` URLs.

The thumbnail host keeps the aspect ratio of the stored still. A portrait URL stays 9:16. A landscape URL stays 16:9. The host may round the requested size to a nearby pixel size.

`thumbnail` and `coverImg` are omitted when the entry has no stored thumbnail. `portrait` is still returned.

`spriteUrl` is separate. It is returned for CloudMatrix on-demand only. See [Sprite previews](./Sprite-Previews.md).

---

## 1. Choose the poster from the response

```javascript
const poster = entry.coverImg?.['720'] ?? entry.thumbnail;

player.load({
  title: entry.name,
  hls: entry.media.hls,
  dash: entry.media.dash,
  poster
});
```

Use `coverImg['360']`, `coverImg['720']`, or `coverImg['1080']`. The URL is already the right shape for that entry.

---

## 2. Size the player from `portrait`

Use `portrait` for the player or poster frame. A `false` value is a 16:9 frame. A `true` value is a 9:16 frame.

```javascript
const frame = entry.portrait
  ? { aspectRatio: '9 / 16' }
  : { aspectRatio: '16 / 9' };
```

---

## Checklist

- Read `portrait` before you choose a player or poster size.
- Use a `coverImg` URL from the response.
- Confirm a landscape entry uses `/width/` and a portrait entry uses `/height/`.
- Confirm a portrait poster stays 9:16.
- Leave `spriteUrl` off live and Media Platform playback. It is CloudMatrix on-demand only.
