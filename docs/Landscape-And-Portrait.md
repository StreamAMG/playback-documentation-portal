# Landscape and Portrait

`GET /v1/entry/{id}` tells you whether a still is landscape or portrait, and returns the poster URL for that shape. Use the URL in the response. Do not build a `/width/` URL yourself.

The Playback SDK and the StreamAMG embed player already read these fields and size the frame. This page covers that behavior, including how to keep a custom portrait size, and how to size a player you create yourself.

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

## 2. Size your own player from `portrait`

Use this when you create the Bitmovin player yourself. The Playback SDK and the embed player already size the frame. See the next section.

A `false` value is a 16:9 frame. A `true` value is a 9:16 frame.

```javascript
const frame = entry.portrait
  ? { aspectRatio: '9 / 16' }
  : { aspectRatio: '16 / 9' };
```

---

## 3. Playback SDK and embed player

### Landscape

The Playback SDK leaves the player element as your page styled it. A custom width or height for a landscape entry stays in place. `portrait: false` and a missing flag are both landscape.

The embed player uses 16:9 when the page has not set a ratio. A ratio already set on the element is kept.

### Portrait

A portrait entry replaces the size of the player element with a 9:16 frame. The SDK writes inline `aspect-ratio`, `width`, `height`, `max-width`, `max-height`, and horizontal margins. The box stays within its parent and is no taller than 90% of the viewport. Those inline styles replace a width or height set in a stylesheet or on the element before play.

A size set on a wrapper around the player is left alone. To keep a portrait player smaller than the default, put it in a narrower parent. It stays inside that parent and keeps the 9:16 ratio.

### Keep your own portrait size

An `!important` rule on the player element wins over the inline frame. Override each property the player writes:

```css
#player {
  width: 360px !important;
  height: 640px !important;
  max-width: none !important;
  max-height: none !important;
  aspect-ratio: auto !important;
  margin-left: 0 !important;
  margin-right: 0 !important;
}
```

A normal stylesheet rule is replaced when the portrait entry loads. Set the override before play. It still applies after the SDK writes the frame.

If a landscape entry plays in that same container later, the SDK puts back the inline size from before the portrait entry.

---

## Checklist

- Read `portrait` before you choose a player or poster size.
- Use a `coverImg` URL from the response.
- Confirm a landscape entry uses `/width/` and a portrait entry uses `/height/`.
- Confirm a portrait poster stays 9:16.
- Leave a landscape SDK player at the size the page set.
- Use `!important` when the page must keep its own portrait size.
- Leave `spriteUrl` off live and Media Platform playback. It is CloudMatrix on-demand only.
