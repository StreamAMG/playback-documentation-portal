# Resume playback — Bitmovin integration

> **Fusion and JWKS clients only**  
> The **`PUT /v1/entry/{entryId}/resume`** flow in this guide is **only** for Playback configurations that use **Fusion** or **JWKS** authentication (as set up by StreamAMG).  
> **CloudPay** clients must **not** implement this HTTP PUT path — resume for CloudPay uses the **legacy** mechanism (e.g. WebSocket / existing SDK resume). Calls to PUT from a CloudPay-only configuration return **403 Forbidden**.

Hub guide for **resume playback** with the StreamAMG **Playback API** and **Bitmovin Player**. Use this page for **prerequisites**, the **shared HTTP contract**, **errors**, and **when to call PUT**. For copy-paste implementations, follow the content-specific tutorials:

| Content | Guide |
|---------|--------|
| **On-demand (VoD)** | [Resume playback — On-demand (VoD) Bitmovin integration](./Resume-Playback-External-Bitmovin-VoD.md) |
| **Live / DVR** | [Resume playback — Live Bitmovin integration](./Resume-Playback-External-Bitmovin-Live.md) |

API reference: [Playback Resume API](../../reference/Playback-Resume-API.yaml) · Main playback read: [Playback API](../../reference/Playback-API.yaml) (`GET` returns **`playFrom`**).

---

## Prerequisites

1. Your Playback site is configured for **Fusion** or **JWKS** authentication — **not CloudPay-only**. Confirm with StreamAMG before building to this guide.
2. **StreamAMG Playback** is integrated (**Playback API base URL** + **`x-api-key`**).
3. Viewers are **authenticated** with a **Fusion** or **JWKS** viewer token (**`Authorization: Bearer`**) that Playback accepts on GET and PUT.
4. **Resume is enabled** on your Playback deployment (StreamAMG / onboarding). If resume is off, **`playFrom`** is omitted and you must **not** call **`PUT …/resume`**.

---

## What resume does

- One saved position per **viewer** + **entry**.
- **GET** `/v1/entry/{entryId}` may return **`playFrom`** when a position exists.
- Your Bitmovin integration **restores** `playFrom` (VoD vs live differs — see tutorials above).
- **PUT** `/v1/entry/{entryId}/resume` **saves** position on pause, end, leave, or destroy.

---

## VoD vs live (pick one tutorial)

The **HTTP contract is the same**; Bitmovin **time semantics** differ.

| | **VoD** | **Live / DVR** |
|---|--------|----------------|
| Typical **`playFrom`** | Seconds from **start** (usually &lt; `1000000000`) | Often **UNIX** on timeline (≥ `1000000000`) |
| Restore in Bitmovin | **`startOffset`** or **`seek`** | **`timeShift`** after source ready |
| Typical **`playTime` on PUT** | `getCurrentTime()` (offset) | `getCurrentTime()` (often UNIX) |
| Typical **`duration` on PUT** | `getDuration()` or GET **`duration`** | **`0`** if player returns **`Infinity`** |

**Rule:** save whatever scale **`getCurrentTime()`** uses for that entry so the next GET **`playFrom`** matches.

---

## Shared flow (both content types)

```mermaid
sequenceDiagram
  participant App
  participant Playback as Playback API
  participant Bitmovin

  App->>Playback: GET /v1/entry/{entryId}
  Playback-->>App: playFrom, media, duration
  App->>Bitmovin: load + restore playFrom
  Note over Bitmovin: VoD seek/startOffset OR live timeShift
  Bitmovin-->>App: viewer watches
  App->>Playback: PUT /v1/entry/{entryId}/resume
  Note over App,Playback: pause / end / leave
```

### Step 1 — GET playback

```http
GET /v1/entry/{entryId}
x-api-key: YOUR_PLAYBACK_API_KEY
Authorization: Bearer YOUR_VIEWER_SESSION_TOKEN
```

- Use the **same `{entryId}`** you will use on PUT (Kaltura id, UUID, etc. — whatever your GET uses for that asset).
- Read **`playFrom`** (optional) and **`media`** for Bitmovin **`load`**.

### Step 2 — Restore in Bitmovin

→ [VoD tutorial](./Resume-Playback-External-Bitmovin-VoD.md#step-2--load-bitmovin-and-restore-playfrom)  
→ [Live tutorial](./Resume-Playback-External-Bitmovin-Live.md#step-2--load-bitmovin-and-restore-with-timeshift)

### Step 3 — PUT resume position

```http
PUT /v1/entry/{entryId}/resume
x-api-key: YOUR_PLAYBACK_API_KEY
Authorization: Bearer YOUR_VIEWER_SESSION_TOKEN
Content-Type: application/json
```

**Same `{entryId}` and same Bearer token as GET.**

Required JSON body:

```json
{
  "playTime": 0,
  "duration": 0,
  "isPlaying": false
}
```

All three fields are **required**. **`playTime`** and **`duration`** must be **numbers** (not `null`). See tutorials for realistic VoD vs live values.

Success is typically **2xx** with an empty body (often **204**).

### When to call PUT

Wire PUT to **viewer actions**, not a tight poll loop:

- **Paused** → `isPlaying: false`
- **Playback finished** → `isPlaying: false`
- **Player destroyed** → PUT before teardown
- **Page hide / tab hidden** → use **`fetch(..., { keepalive: true })`** if the page may unload

Full event wiring is in the **VoD** and **Live** example pages.

---

## StreamAMG embed player

If you use **`playbackembedplayer.js`**, resume GET/PUT for Bitmovin is built in when resume is enabled and a viewer token is present. Custom Bitmovin integrations should follow the VoD or Live tutorial.

---

## Error handling

| Status | Likely cause |
|--------|----------------|
| **400** | Invalid body (`null`/`NaN`), resume not configured, or **entry id mismatch** on PUT vs authenticated context |
| **401** | Missing or invalid Bearer token |
| **403** | **Not Fusion/JWKS** (e.g. CloudPay-only) — message: *Resume writes are only supported for Fusion and JWKS configurations* |
| **404** | Entry not found for this request |

**Do not block playback** if PUT fails. Common bug: **`duration: null`** from `JSON.stringify({ duration: Infinity })` on live — send **`0`**.

---

## Checklist (all integrators)

- [ ] Playback auth is **Fusion** or **JWKS** (not CloudPay-only for this PUT)
- [ ] GET with **`x-api-key`** + viewer **Bearer**
- [ ] Resume **enabled** on your deployment
- [ ] Implemented **VoD** or **Live** tutorial (not both mixed in one code path)
- [ ] Same **`entryId`** + token on GET and PUT
- [ ] Finite **`duration`** on every PUT

---

## Internal reference (StreamAMG staff)

- [Resume playback — internal workflow](./Resume-Playback-Internal.md)
