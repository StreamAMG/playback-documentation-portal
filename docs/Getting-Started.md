# Getting Started

This page helps you **start a new Playback integration** — what you need from StreamAMG, your first API call, and where to go next.

---

## Before you write code

Confirm with StreamAMG (client delivery / onboarding):

| Item | Why you need it |
|------|------------------|
| **Playback API base URL** | e.g. production `https://api.playback.streamamg.com` — staging/dev host if applicable |
| **`x-api-key`** | Identifies your Playback client configuration on every request |
| **Auth model** | **Fusion**, **CloudPay**, **JWKS**, or external JWT — drives entitlements and **which resume mechanism** you use |
| **Entry identifiers** | What your CMS sends as `{id}` (Kaltura entry id, CloudMatrix UUID, etc.) — must be **consistent** across GET and resume PUT |
| **Bitmovin license key** | Required for Bitmovin Player (embed supplies via `data-bitmovin-license-key`) |

---

## Choose your integration path

### Path A — Playback API + your own Bitmovin player

**Recommended** for product teams building a custom player experience.

1. Call **`GET /v1/entry/{entryId}`** with `x-api-key` and, when required, `Authorization: Bearer {viewerToken}`.
2. Pass `media` (e.g. HLS URL) into [Bitmovin `Player.load`](https://developer.bitmovin.com/playback/docs/getting-started-with-the-web-player).
3. If the response includes **`playFrom`**, restore position per content type ([VoD](./resume/Resume-Playback-External-Bitmovin-VoD.md) vs [Live](./resume/Resume-Playback-External-Bitmovin-Live.md)).
4. On pause / leave, **`PUT /v1/entry/{entryId}/resume`** — **Fusion and JWKS only** ([Resume hub](./resume/Resume-Playback-External-Bitmovin.md)).

References: [Playback API](../reference/Playback-API.yaml) · [Playback Resume API](../reference/Playback-Resume-API.yaml)

### Path B — Embed player

**Fastest** for pages and CMS-driven sites.

1. Include the embed script and a `streamamg-embed` container ([Embed Player Setup](./embed-player/Embed-Player.md)).
2. Set `data-entry-id`, `data-playback-api-key`, `data-bitmovin-license-key`.
3. Store the viewer token in `localStorage` (default key `streamamg_auth_token`) after login.

Resume and Playback calls are handled inside the embed when enabled on your tenant.

CMS guides: [WordPress](./embed-player/WordPress.md) · [Drupal](./embed-player/Drupal.md) · [Contentful](./embed-player/Contentful.md)

### Path C — Playback JavaScript SDK (**DEPRECATED Summer 2025**)

The SDK (`playback.js`) wraps API + Bitmovin for older integrations.

> **New projects:** StreamAMG advises integrating via the **Playback API** directly rather than the SDK. See [SDK General](./SDK/General.md).

Production script URL:

```
https://sdk.playback.streamamg.com/v1/playback.js
```

Staging (when using staging API keys):

```
https://sdk.playback.staging.streamamg.com/v1/playback.js
```

---

## Your first Playback API call

Replace placeholders with values from StreamAMG.

```bash
curl -sS \
  -H "x-api-key: YOUR_PLAYBACK_API_KEY" \
  -H "Authorization: Bearer YOUR_VIEWER_TOKEN" \
  "https://api.playback.example.com/v1/entry/0_abc123"
```

- Omit the **`Authorization`** header only for **free-to-watch** entries.
- A successful response includes **`media`** (manifest URLs) and fields your player needs (`duration`, `playFrom`, adverts, etc.) — see [Playback API](../reference/Playback-API.yaml).

**Common HTTP outcomes**

| Status | Typical cause |
|--------|----------------|
| **401** | Missing/invalid API key or Bearer token |
| **403** | Viewer not entitled, geo blocked, or session limit |
| **404** | Unknown entry id for this configuration |

---

## Documentation map (by goal)

| Goal | Read |
|------|------|
| Understand the platform | [Introduction](./Introduction.md) |
| Adverts, resume, analytics, geo | [Player Integrations](./Player-Integrations.md) |
| Custom Bitmovin + resume (VoD) | [Resume — VoD](./resume/Resume-Playback-External-Bitmovin-VoD.md) |
| Custom Bitmovin + resume (live) | [Resume — Live](./resume/Resume-Playback-External-Bitmovin-Live.md) |
| Embed + authentication | [Embed Player](./embed-player/Embed-Player.md) |
| Bitmovin UI styling | [Video Player Configuration](./Video-Players.md) |
| Geo setup (CloudMatrix) | [Geo Restrictions](./Geo-Restrictions-External.md) |
| SDK errors | [Error Handling](./SDK/Error-Handling.md) |
| API contracts | Sidebar → **API** (OpenAPI) |

---

## Environments

| Environment | Notes |
|-------------|--------|
| **Production** | Production API key + production API host |
| **Staging / dev** | Separate keys and hosts from StreamAMG — do not mix with production |

Exact URLs are issued per client during onboarding.

---

## Getting help

- **Configuration or feature enablement** (resume, ads, new tenant): contact StreamAMG client delivery — see internal [Client Onboarding](./Client-Onboarding.md) if you are StreamAMG staff.
- **Integration behaviour**: use the guides above; for resume issues, confirm **Fusion/JWKS** before implementing HTTP PUT.
