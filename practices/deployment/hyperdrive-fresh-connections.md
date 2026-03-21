---
concern: deployment
tech: [cloudflare, postgres, workers, hyperdrive]
priority: foundational
source-repo: wellforce-design-system
applies-to: [cloudflare, hyperdrive]
---
# Cloudflare Hyperdrive: Fresh database connections per request

## PATTERN
When using Cloudflare Hyperdrive for database connection pooling, create a fresh database client instance per request. Never cache the client at module scope. Hyperdrive proxy URLs are request-scoped -- they become invalid after the request that created them completes. Hyperdrive handles connection pooling at the proxy level, so client-side pooling is unnecessary.

## WHY
Hyperdrive gives each request a proxy URL that routes to a pooled connection on Cloudflare's side. Module scope in Workers persists across requests within the same isolate, so a cached client from request 1 holds a reference to a dead proxy endpoint by request 2. This causes 500 errors on all database operations after the first request -- an intermittent failure that's difficult to reproduce locally.

## EXAMPLE
```typescript
// lib/db.ts -- correct pattern for Hyperdrive
import postgres from "postgres";

export function getDb(url: string) {
  return postgres(url, {
    prepare: false,    // required: Hyperdrive doesn't support prepared statements
    fetch_types: false, // skip pg_catalog queries over Hyperdrive
    max: 1,            // single connection per request
    idle_timeout: 20,  // clean up if idle
  });
}

// In a server action or API route:
export async function getMyData() {
  const db = getDb(process.env.DATABASE_URL!);
  try {
    return await db`SELECT * FROM my_table`;
  } finally {
    await db.end();
  }
}
```

## CHECK
How to verify if a repo already follows this:
- [ ] No module-scope `let db = ...` or `const db = ...` for postgres/database clients
- [ ] Database client creation happens inside request handlers, not at import time
- [ ] `prepare: false` is set when using Hyperdrive
- [ ] No comments claiming "module scope is per-request in Workers" (this is wrong)

## IMPLEMENT
1. Move database client creation from module scope into a factory function
2. Call the factory function inside each server action or API route handler
3. Set `prepare: false` and `fetch_types: false` for postgres.js with Hyperdrive
4. Ensure connections are cleaned up (`.end()`) after use
5. Remove any module-scope connection caching logic

## NOTES
- This is FOUNDATIONAL for any project using Hyperdrive -- not following this pattern causes intermittent 500s that are nearly impossible to debug without understanding the proxy URL lifecycle.
- Local development (without Hyperdrive) may work fine with cached connections, making this bug invisible until deployment.
- If using Drizzle ORM, the same rule applies -- create the drizzle instance per request wrapping a fresh postgres client.
