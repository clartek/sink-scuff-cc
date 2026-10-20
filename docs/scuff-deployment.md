# scuff.cc deployment

This fork tracks upstream Sink and deploys as the existing Cloudflare Worker `sink` on `scuff.cc`.

## Production resources

Preserve the existing bindings and runtime secrets. Never delete the original KV namespace during upgrades.

- `KV`: namespace `82489f4c8d4148a68569d7d41fc25450` (`Sink-scuffed`)
- `DB`: D1 database `sink-scuff-cc`, ID `826b373c-ad18-4a53-81da-3642a24a8e26`
- `ANALYTICS`: dataset `sink_events`
- `sink_engine`: legacy alias for the same dataset
- `R2`: bucket `sink`
- `AI`: existing Workers AI binding
- `ASSETS`: generated production assets

The D1 ID in tracked `wrangler.jsonc` is an upstream local/development placeholder. Production deployments must generate `wrangler.deploy.jsonc` with the variables below. Do not deploy the placeholder directly.

## Build and deploy

Use Node.js 24+ and pnpm 11.11.0. Configure these deployment variables in a protected local environment or Workers Builds:

```dotenv
DEPLOY_D1_DATABASE_ID=826b373c-ad18-4a53-81da-3642a24a8e26
DEPLOY_D1_DATABASE_NAME=sink-scuff-cc
DEPLOY_KV_NAMESPACE_ID=82489f4c8d4148a68569d7d41fc25450
DEPLOY_R2_BUCKET_NAME=sink
DEPLOY_ANALYTICS_DATASET=sink_events
```

Wrangler also needs `CLOUDFLARE_API_TOKEN` and `CLOUDFLARE_ACCOUNT_ID`. Never commit these credentials. Keep the existing `NUXT_SITE_TOKEN`, `NUXT_CF_ACCOUNT_ID`, and `NUXT_CF_API_TOKEN` runtime values unchanged. `NUXT_DATASET=sink_events` is tracked in Worker vars to keep analytics reads aligned with writes.

```bash
pnpm install --frozen-lockfile
pnpm lint
pnpm types:check
pnpm build
pnpm deploy:worker
```

The deploy command applies D1 schema migrations before uploading the Worker. The existing daily backup schedule remains configured in `wrangler.jsonc`.

## Tests

For local tests only, create an ignored `.env` with a non-production `NUXT_SITE_TOKEN`, then run `pnpm test --run` after building. The test pool disables remote bindings so it cannot access production storage or require Cloudflare login. Remove the local test `.env` before a production build; production must use the existing runtime token, not a baked-in local test default.

## Storage migration and verification

The initial KV-to-D1 migration copied both existing links without errors. Future ordinary deployments do not require a forced migration. Do not force-rescan or replace the original KV namespace unnecessarily.

After updates, verify `https://scuff.cc/`, `/dashboard`, authenticated `/api/verify`, existing short-link redirects, analytics, and create/edit/delete of a disposable test link. Keep a secure backup outside Git before storage changes.
