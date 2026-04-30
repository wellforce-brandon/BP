---
concern: database
priority: recommended
tags: [drizzle-orm, drizzle-kit, migrations, backfill, postgres]
---
# Drizzle-Kit `--custom` Workflow for Data Backfill + NOT NULL Flip

## CONTEXT

Drizzle-kit's auto-generator can emit DDL (`CREATE TABLE`, `ALTER TABLE ADD COLUMN`, etc.) but cannot synthesize data backfills. A common multi-step migration -- add nullable column -> backfill rows -> flip to NOT NULL -- needs hand-written SQL for the middle step. The `--custom` flag is the canonical path. The combined workflow below avoids two common pitfalls: (a) `pnpm db:generate` re-emitting the NOT NULL ALTER as a *separate* migration after the custom one runs, and (b) the journal `when`-timestamp silent-skip pattern.

## PATTERN

### Workflow

1. **Update the schema file first** -- flip the column to `.notNull()` in your Drizzle schema (e.g., `partnerId: text("partnerId").notNull().references(...)`).
2. **Generate the custom stub** -- `pnpm exec drizzle-kit generate --custom --name=<phase>_<description>`. This produces an empty `.sql` file under `drizzle/` AND updates the snapshot under `drizzle/meta/` to match the new schema state. The empty SQL file is intentional -- `--custom` always emits an empty stub regardless of detected diff.
3. **Hand-write the SQL** in the generated file. Order matters; follow the parent -> child dependency order:
   ```sql
   -- 1. Insert parent rows referenced by the FK target
   INSERT INTO "parent_table" (...) VALUES (...);
   --> statement-breakpoint

   -- 2. Backfill the new column on existing rows
   UPDATE "child_table" SET "parent_id" = 'seed-id' WHERE "parent_id" IS NULL;
   --> statement-breakpoint

   -- 3. (optional) Backfill any sibling tables that share the FK
   UPDATE "app_logs" SET "parent_id" = 'seed-id' WHERE "parent_id" IS NULL;
   --> statement-breakpoint

   -- 4. Seed any associative rows (memberships, etc.)
   INSERT INTO "associations" (...) VALUES (...);
   --> statement-breakpoint

   -- 5. Flip the column to NOT NULL last, after every row is filled
   ALTER TABLE "child_table" ALTER COLUMN "parent_id" SET NOT NULL;
   ```
4. **Verify the journal `when` is monotonic** -- open `drizzle/meta/_journal.json`, find the new entry (highest `idx`), and confirm `when > max(prior when)`. Drizzle-kit assigns `Date.now()` on generate; if your clock is behind a previous entry's `when` (or any prior entry was hand-bumped to a future date), the new migration will be silently skipped at apply time. Hand-bump if needed (e.g., `1777593600005` if the prior max was `1777593600004`). See LL-G `kb/typescript/drizzle-kit-migrate-silent-skip.md`.
5. **Apply** -- `pnpm db:migrate` (with `DATABASE_URL` set to the public proxy URL when running locally against a hosted DB; PgBouncer's internal hostname won't resolve outside the platform's network).
6. **Verify on the target DB** -- `psql "$DATABASE_PUBLIC_URL" -c "SELECT count(*) FROM drizzle.__drizzle_migrations"` should show the new row, and a quick `\d child_table` should confirm the column is `NOT NULL`. Drizzle-kit's "applied successfully" message lies on silent-skips; trust the row count.

### Why this order

- **Schema-first, custom second** keeps the snapshot honest: the next `drizzle-kit generate` will not detect a "missing NOT NULL ALTER" and emit a duplicate migration, because the snapshot already reflects NOT NULL.
- **Backfill before NOT NULL** is mandatory -- flipping NOT NULL on a column that still has any NULL row aborts the whole transaction.
- **Statement breakpoints** (`--> statement-breakpoint`) tell drizzle-kit to send each statement separately; without them, some Postgres DDL forms fail when batched with DML in a single round-trip.

## ANTI-PATTERN

```sql
-- Backfill and NOT NULL in the same statement-less file.
-- Without statement-breakpoints, the ALTER may run before the UPDATE commits.
UPDATE child_table SET parent_id = 'X' WHERE parent_id IS NULL;
ALTER TABLE child_table ALTER COLUMN parent_id SET NOT NULL;
```

```ts
// Updating the schema AFTER hand-writing the custom SQL.
// drizzle-kit detects the NOT NULL diff and emits an extra (now redundant) migration.
```

## NOTES

- Use stable readable IDs for singleton seeds (`partner_socium` instead of cuid2). Grep-friendly in production logs and SQL spelunking; the `id()` cuid2 default is bypassed by an explicit `INSERT ... VALUES (...)`.
- For very large backfills (millions of rows), break the UPDATE into chunked batches in the SQL or use a deferred batch-update worker -- but for one-shot migrations under ~1M rows, a single UPDATE with a covered index is fine.
- This workflow generalizes beyond NOT NULL flips -- any "DDL + custom DML in one migration" follows the same pattern.

## SOURCES

- Surfaced and documented during the multi-broker Phase 1b backfill in a Vigilis-style codebase; migration `0035_phase_1b_socium_backfill.sql`.
- Companion gotcha: `LL-G/kb/typescript/drizzle-kit-migrate-silent-skip.md`.
