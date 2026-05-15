# Introduction

**StreamAMG Playback** is a video delivery service that sits between your application and your media origin (for example Kaltura / Media Platform and CloudMatrix). It authenticates viewers, applies business rules (entitlements, geo, sessions), and returns **playback-ready data** — signed manifests, metadata, optional **`playFrom`** resume position, adverts, and configuration your player needs.

This portal documents how **you** integrate Playback into web and mobile experiences. StreamAMG configures your tenant (players, features, auth model) via onboarding; your team implements the client using the **Playback API** and/or the **embed player**.

---

## What Playback provides

| Capability | Summary |
|------------|---------|
| **Playback API** | `GET /v1/entry/{id}` — manifests, metadata, entitlements outcome, resume read (`playFrom`), ads, and related fields for a single entry. |
| **Resume API** | `PUT /v1/entry/{id}/resume` — save viewer position (**Fusion and JWKS** sites only; see [Resume](./resume/Resume-Playback-External-Bitmovin.md)). |
| **Embed player** | Drop-in Bitmovin-based player for sites and CMS ([Embed Player](./embed-player/Embed-Player.md)). |
| **Playback SDK (JavaScript)** | Legacy all-in-one JS integration — **new builds should prefer the Playback API** ([SDK note](./SDK/General.md)). |

Supported player for new integrations: **[Bitmovin Player](https://bitmovin.com/video-player)**. Playback returns media URLs and rules; Bitmovin plays the stream.

---

## How viewers access content

Playback classifies content by access level. Your app must send the right headers on **`GET /v1/entry/{id}`**:

| Access level | `x-api-key` | `Authorization: Bearer` |
|--------------|-------------|------------------------|
| **Free-to-watch** | Required | Not required |
| **Freemium** (logged in) | Required | Required |
| **Premium** (subscribed / entitled) | Required | Required |

Entitlements are enforced using the auth model configured for your site (**Fusion**, **CloudPay**, **JWKS**, or external JWT enrichment). See [Getting Started](./Getting-Started.md) and the [Playback API](../reference/Playback-API.yaml) reference.

---

## Integration paths (choose one)

| Path | Best for | You implement |
|------|----------|----------------|
| **[Playback API + your Bitmovin](./Getting-Started.md#path-a--playback-api--your-own-bitmovin-player)** | Custom web/apps, full UI control | GET playback, Bitmovin `load` / events, resume PUT (Fusion/JWKS) |
| **[Embed player](./embed-player/Embed-Player.md)** | Marketing sites, CMS (WordPress, Drupal, Contentful) | Script tag, `data-*` attributes, auth token in `localStorage` |
| **[Playback SDK](./SDK/General.md)** | Existing SDK integrations only | `playback.js` initialise + play — not recommended for new projects **DEPRECATED** |

---

## Optional platform features

Features are **enabled per tenant** during StreamAMG onboarding (not self-service in this portal):

- **Resume** — continue watching across sessions ([Resume hub](./resume/Resume-Playback-External-Bitmovin.md))
- **Global adverts** — pre/mid/post rolls via configuration ([Player Integrations](./Player-Integrations.md#global-adverts))
- **Analytics** — typically Bitmovin Analytics (MUX may exist on legacy configs)
- **Geo restrictions** — country/region rules via CloudMatrix + Playback ([Geo Restrictions](./geo/Geo-Restrictions-External.md))
- **External entitlements** — JWKS or JWT enrichment ([JWKS](./jwks/External-Entitlements-JWKS.md), [JWT enrichment](./jwks/External-Entitlements-JWT-Enrichment.md))
- **Multiview / playlists** — SDK-oriented layouts ([Multiview](./SDK/Multiview.md), [Playlist](./SDK/Playlist.md))

---

## Security and operations

- All API calls use your site **`x-api-key`**; protected content also requires a **viewer Bearer token** your platform issues after login.
- Manifest URLs returned by Playback are intended for player consumption; treat API keys and tokens as secrets.
- Use **staging** credentials and endpoints provided by StreamAMG for non-production testing (do not use production keys on staging URLs).

---

## Next steps

1. Read **[Getting Started](./Getting-Started.md)** — credentials, first API call, and which guide to open next.  
2. Skim **[Player Integrations](./Player-Integrations.md)** — adverts, resume, analytics, geo, and Bitmovin.  
3. Use the **API** section in the sidebar for OpenAPI references.

For StreamAMG staff: internal onboarding and config examples remain under **Client Onboarding** and **Client Playback Examples** in this portal.
