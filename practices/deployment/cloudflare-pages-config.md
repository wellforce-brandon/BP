---
concern: deployment
tech: [cloudflare, nextjs, typescript]
priority: recommended
source-repo: bp-website
applies-to: [nextjs, cloudflare]
---
# Cloudflare Workers/Pages Deployment with OpenNext

## PATTERN
Deploy Next.js apps to Cloudflare Workers using the `@opennextjs/cloudflare` adapter. Use Doppler for secrets management, syncing to Cloudflare's encrypted store. Build without Turbopack (`next build` not `next build --turbo`) since OpenNext requires the standard compiler.

## WHY
Cloudflare Workers provides edge deployment with zero cold starts, global distribution, and no server management. The OpenNext adapter translates Next.js server components and API routes to Workers-compatible code. Doppler centralizes secrets across environments (dev/stg/prd) and syncs to Cloudflare's encrypted store.

## EXAMPLE
From bp-website (`open-next.config.ts`):
```typescript
import { defineCloudflareConfig } from "@opennextjs/cloudflare";
export default defineCloudflareConfig({});
```

Build commands:
```bash
npm run build     # Production build (no --turbo, OpenNext requires standard compiler)
npm run preview   # Build + preview on local Cloudflare Workers
npm run deploy    # Build + deploy to Cloudflare Workers
```

Secrets sync from Doppler to Cloudflare:
```bash
doppler secrets --json --project bp-website --config prd \
  | jq -c 'with_entries(.value = .value.computed)' \
  | CLOUDFLARE_ACCOUNT_ID=<account-id> wrangler secret bulk
```

## CHECK
How to verify if a repo already follows this:
- [ ] `@opennextjs/cloudflare` is in dependencies
- [ ] `open-next.config.ts` exists
- [ ] Build script does not use `--turbo` flag
- [ ] Doppler project exists for this repo (or secrets are managed centrally)
- [ ] `wrangler.toml` or equivalent Cloudflare config exists

## IMPLEMENT
1. Install `@opennextjs/cloudflare` as a dependency
2. Create `open-next.config.ts` with `defineCloudflareConfig({})`
3. Update build scripts to remove `--turbo` flag
4. Set up Doppler project with dev/stg/prd configs
5. Configure Cloudflare git integration or manual deploy pipeline
6. Sync secrets: `doppler secrets --json | wrangler secret bulk`

## NOTES
- Never commit real secrets -- Cloudflare deploys via git integration; secrets persist in their encrypted store
- `.env.local` is the fallback for local development when Doppler is unavailable
- Use `doppler run` to inject secrets into local dev server
