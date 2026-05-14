---
internal: true
---

# Resume playback — internal workflow

This document describes how **resume playback** works inside the Playback stack for engineers operating or extending the system. For integrator-facing steps and Bitmovin wiring, see **Resume playback — Bitmovin integration** (external guide).

---

## Purpose

- Persist **last watched position** per **tenant + authenticated customer + entry**.
- **GET** `/v1/entry/{videoId}` returns **`playFrom`** when resume is enabled and a stored position exists. The stored value is whatever the client last wrote as **`playTime`** (typically **VoD**: seconds from start; **live / DVR**: UNIX timeline seconds — see the external Bitmovin guide).
- **PUT** `/v1/entry/{entryId}/resume` updates that position for **Fusion** and **JWKS** clients only, using the **same authentication and entitlement pipeline as GET** (no separate resume JWT layer).

---

## Configuration (Playback client config)

Resume is gated by the Bitmovin integration block (internal name; do not expose schema detail to external integrators):

- `player.bitmovin.integrations.resume.enabled` must be `true` for resume **read** on GET and for **write** on PUT (Fusion/JWKS).
- Additional keys may exist for legacy or signing uses; **HTTP PUT resume** does not require a dedicated resume token in `Authorization`.

External documentation should only state that **resume must be enabled on the customer’s Playback setup** (StreamAMG / onboarding).

---

## Authentication matrix

| Auth model | GET `playFrom` source | PUT `/…/resume` |
|------------|------------------------|-----------------|
| **Fusion** or **JWKS** (`isPlaybackResumeAuthentication`) | **New** Playback resume table (`DDB_PLAYBACK_RESUME_TABLE`), key `{tenantId}-{customerId}-{entryId}` | **Allowed** — writes same table |
| **CloudPay** (and not JWKS) | **Legacy** CloudPay resume table (`DDB_RESUME_TABLE`), key `customerId#entryId` style | **Not supported** — returns **403**; CloudPay continues to use existing **non–HTTP-PUT** resume path (e.g. WebSocket / legacy SDK flow) |

---

## DynamoDB

| Table (env-driven) | Used for | Written by PUT? |
|--------------------|----------|------------------|
| `DDB_PLAYBACK_RESUME_TABLE` | Fusion/JWKS resume rows | **Yes** (PUT handler only, when allowed) |
| `DDB_RESUME_TABLE` | CloudPay legacy resume | **No** from new PUT endpoint |

Item shape for the new table (conceptual): composite `id`, `entryId`, `playTime`, `duration`, `isPlaying`, timestamps, etc., as implemented in the Lambda.

---

## GET playback sequence (relevant steps)

1. **Authenticate** (`AuthenticationService`).
2. Load **playback store entity**; **geo** and **entitlements**; **CloudPay session** validation when applicable.
3. **`ResumeService.handleResume`**:
   - If no **customer** or resume **disabled** → skip `playFrom`.
   - If **Fusion/JWKS** → `getPlaybackResumePlayTimeFor` / `buildPlaybackResumeId(tenantId, customerId, computedVideoId)`.
   - If **CloudPay** → `getResumePlayTimeFor` (legacy table).
4. **Media processing**, tokenisation, ads — unchanged.

There is **no** requirement to emit a resume-only JWT header on GET for the new HTTP PUT flow.

---

## PUT resume sequence

1. **`buildAuthorizedPlaybackContext`** (controller private helper) mirrors GET **through session validation**:

   - `authenticate` → `getPlaybackStoreEntity` → `processPlaybackStoreEntity` → **geo** → **`checkEntitlement`** → **`validateSession`**.

2. **`ResumeService.updateResume`**:

   - Throws **403** if not **Fusion/JWKS**.
   - Requires **resume.enabled** on Bitmovin integration.
   - Requires **customer** on context (from auth).
   - Requires **`context.computedVideoId === route entryId`** (path param must match the resolved entry id after store processing — same rule as when validating a scoped token).
   - **`putPlaybackResumeForEntry`** → new table.

This intentionally reuses the **same authn/authz/session** behaviour as GET so API Gateway and clients only need the **normal user/session `Authorization: Bearer`** (plus `x-api-key`), not a second token type for PUT.

---

## API Gateway / routing

- **PUT** `/v1/entry/{entryId}/resume` is deployed as a first-class route on the Playback API (same API key model as GET where configured).
- **Do not** attach **AWS_IAM** (SigV4) to this method if clients send **Bearer** JWT in `Authorization`; IAM expects a different `Authorization` format.

---

## Embed / SDK notes (internal)

- **Embed** (`playbackembedplayer.js`): resume **PUT** should send the **same user auth token** used for GET, not a separate header-only resume secret.
- **Playback SDK** may still use **WebSocket** resume for CloudPay; new **HTTP PUT** path is **Fusion/JWKS** only.

---

## Quick troubleshooting

| Symptom | Likely cause |
|---------|----------------|
| PUT **403** “Fusion and JWKS” | CloudPay-only client hitting new PUT |
| PUT **400** entry mismatch | Path `entryId` ≠ `computedVideoId` after GET-style resolution (e.g. wrong id vs Kaltura entry id) |
| **SigV4 / Authorization** errors at edge | API Gateway method auth or client tool sending AWS Signature on `Authorization` |
| GET never shows `playFrom` | Resume disabled, user not authenticated, or no row in the applicable table |

---

## Related external doc

- [Resume playback — Bitmovin integration](./Resume-Playback-External-Bitmovin.md)
