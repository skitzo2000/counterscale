# Amnesia Labs deployment notes

This fork tracks upstream `benvinegar/counterscale`. The only divergence is
this file — all code matches upstream so we can pull upstream updates cleanly.

## Cloudflare Workers Builds configuration

Build runs at the repo root so turbo orchestrates the tracker build and copy
step before the server build. Running from `packages/server` skips
`copytracker`, which leaves `/tracker.js` unavailable at runtime.

| Field | Value |
|-------|-------|
| Root directory | `/` (repo root) |
| Build command | `pnpm install && pnpm run build` |
| Deploy command | `pnpm --filter @counterscale/server exec wrangler deploy` |

## Worker environment

Set in CF Dashboard → Workers & Pages → counterscale → Settings → Variables
and Secrets:

| Var | Type | Notes |
|-----|------|-------|
| `CF_ACCOUNT_ID` | Plaintext | Amnesia Labs CF account ID |
| `CF_BEARER_TOKEN` | Secret | API token with `Account Analytics: Read` |
| `CF_AUTH_ENABLED` | Plaintext | `false` — we gate with CF Access + Keycloak |
| `CF_STORAGE_ENABLED` | Plaintext | `true` — daily rollups into the R2 bucket below |

## Bindings (already in `packages/server/wrangler.json`)

- Analytics Engine dataset: `metricsDataset`
- R2 bucket: `counterscale-daily-rollups` (+ `-dev` preview)
- Assets: `./build/client`

## Access gating

Served from `analytics.amnesia-labs.com`. Cloudflare Access applications
(on the `amnesia-labs` Zero Trust team):

| Path | Policy | Identity |
|------|--------|----------|
| `/tracker.js` | Bypass (Everyone) | — |
| `/collect` | Bypass (Everyone) | — |
| `/dashboard*` | Allow | OIDC claim `groups` ∋ `analytics` via Keycloak |

Keycloak realm: `amnesia-labs` on `auth.amnesia-labs.com`. OIDC client:
`cloudflare-access`.

## Tracker embed

Used on amnesia-labs.com pages:

```html
<script
    id="counterscale-script"
    data-site-id="amnesia-labs"
    src="https://analytics.amnesia-labs.com/tracker.js"
    defer
></script>
```
