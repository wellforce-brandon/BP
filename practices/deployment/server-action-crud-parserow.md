---
concern: deployment
tech: [nextjs, postgres, server-actions, cloudflare]
priority: recommended
source-repo: wellforce-design-system
applies-to: [nextjs, postgres, cloudflare-workers]
---
# Server Action CRUD with parseRow and ALLOWED_FIELDS

## PATTERN
Standardize database CRUD server actions with three components:
1. **ALLOWED_FIELDS set** -- whitelist of columns that can be updated, preventing field injection
2. **parseRow() helper** -- normalizes JSONB columns (which arrive as strings when using postgres.js with `fetch_types: false`) into proper objects
3. **Auth gating** -- every mutation checks `getAuthUser()` and returns null/empty early if not authenticated

## WHY
This pattern prevents three common bugs simultaneously:
- **Field injection**: Without an allowlist, callers could pass `{ user_id: 'attacker' }` in the updates object and overwrite ownership.
- **JSONB parsing**: postgres.js with `fetch_types: false` (required on Cloudflare Workers/Hyperdrive) returns JSONB as strings. Without `parseRow()`, downstream code silently gets `undefined` when accessing nested properties.
- **Silent auth failures**: Server actions that throw on auth failure produce opaque 500 errors. Returning null/empty with a `console.warn` is debuggable and safe.

## EXAMPLE
```typescript
'use server';

import { query } from '@/lib/db/pool';
import { getAuthUser } from '@/lib/db/get-user';
import type { SavedStyle, StyleSeeds } from './types';

const ALLOWED_FIELDS = new Set(['name', 'seeds']);

function parseRow(row: Record<string, unknown>): SavedStyle {
  const seeds = typeof row.seeds === 'string'
    ? JSON.parse(row.seeds) : row.seeds;
  return { ...row, seeds } as SavedStyle;
}

export async function getMyStyles(): Promise<SavedStyle[]> {
  const user = await getAuthUser();
  if (!user) { console.warn('[styles] no auth user'); return []; }
  const { rows } = await query(
    'SELECT * FROM styles WHERE user_id = $1 ORDER BY updated_at DESC',
    [user.id],
  );
  return rows.map((r) => parseRow(r as Record<string, unknown>));
}

export async function updateStyle(
  id: string,
  updates: Partial<{ name: string; seeds: StyleSeeds }>,
): Promise<SavedStyle | null> {
  const user = await getAuthUser();
  if (!user) return null;

  const fields: string[] = [];
  const values: unknown[] = [];
  let idx = 1;

  for (const [key, value] of Object.entries(updates)) {
    if (!ALLOWED_FIELDS.has(key)) continue;
    fields.push(`${key} = $${idx++}`);
    values.push(key === 'seeds' ? JSON.stringify(value) : value);
  }
  if (fields.length === 0) return null;

  fields.push(`updated_at = $${idx++}`);
  values.push(new Date().toISOString());
  values.push(id);
  const idIdx = idx++;
  values.push(user.id);
  const userIdx = idx++;

  const { rows } = await query(
    `UPDATE styles SET ${fields.join(', ')}
     WHERE id = $${idIdx} AND user_id = $${userIdx} RETURNING *`,
    values,
  );
  return rows[0] ? parseRow(rows[0] as Record<string, unknown>) : null;
}
```

## CHECK
How to verify if a repo already follows this:
- [ ] Every server action that writes to the database uses an ALLOWED_FIELDS whitelist
- [ ] Every server action that reads JSONB columns has a parseRow() helper
- [ ] Every mutating server action checks auth and returns null/empty instead of throwing

## IMPLEMENT
Steps to adopt this:
1. Create one CRUD file per database table (e.g., `lib/styles/db.ts`)
2. Define `ALLOWED_FIELDS` as a `Set` at module scope
3. Write a `parseRow()` function that handles all JSONB columns for that table
4. Gate every exported function with `getAuthUser()`, returning null/empty on failure
5. Add `console.warn` with the function name when auth fails for server-side debugging

## NOTES
- The `ALLOWED_FIELDS` set should NEVER include `id`, `user_id`, `created_at`, or `updated_at` -- these are system-managed.
- The `parseRow()` pattern is only needed when using postgres.js with `fetch_types: false`. If using an ORM like Drizzle or Prisma, JSONB is parsed automatically.
- Auto-discovered from wellforce-design-system style persistence implementation.
