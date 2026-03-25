---
concern: error-handling
tech: [drizzle, sqlite, typescript]
priority: recommended
source-repo: RepoTracker
applies-to: [typescript, sqlite]
---
# Single Source of Truth for Database Schema

## PATTERN
Define the database schema in exactly one place -- the ORM schema file. Generate all migrations, init scripts, and type definitions from that single source. Never hand-write CREATE TABLE statements that duplicate the ORM definition.

## WHY
When schema is defined in both an ORM (e.g., Drizzle `sqliteTable()`) and raw SQL (e.g., `CREATE TABLE IF NOT EXISTS`), any column addition, rename, or default change must be updated in both places. They **will** drift. The ORM schema and the actual database diverge silently, causing:
- Missing columns at runtime (ORM expects a column the DB doesn't have)
- Wrong defaults (ORM says `default("main")`, SQL says `DEFAULT 'master'`)
- Type mismatches (ORM says boolean mode, SQL says plain INTEGER)

## EXAMPLE
From RepoTracker -- the anti-pattern:

`src/db/schema.ts` (ORM definition):
```typescript
export const repository = sqliteTable("repository", {
  id: integer("id").primaryKey({ autoIncrement: true }),
  name: text("name").notNull(),
  localPath: text("local_path"),
  // ... 5 more columns
});
```

`src/db/init.ts` (duplicated as raw SQL):
```sql
CREATE TABLE IF NOT EXISTS repository (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  name TEXT NOT NULL,
  local_path TEXT,
  -- ... same 5 columns, manually kept in sync
);
```

The fix -- use Drizzle's migration tooling:
```bash
pnpm drizzle-kit generate  # generates SQL from schema.ts
pnpm drizzle-kit migrate   # applies migrations
```

Or for simple apps, use Drizzle's `migrate()` function at startup instead of hand-written init SQL.

## CHECK
- [ ] Search for `CREATE TABLE` in non-migration files (init scripts, seed files)
- [ ] Compare ORM schema columns against any raw SQL schema definitions
- [ ] Verify migration workflow generates SQL from the ORM, not the other way around

## IMPLEMENT
1. Delete the hand-written `CREATE TABLE` init script
2. Configure `drizzle-kit` with `schema` and `out` paths in `drizzle.config.ts`
3. Run `drizzle-kit generate` to create the initial migration from the ORM schema
4. Execute migrations at app startup (Drizzle's `migrate()` or run SQL files via plugin-sql)
5. For future changes, modify only `schema.ts` and regenerate

## NOTES
- This applies to any ORM: Drizzle, Prisma, TypeORM, Diesel, SQLAlchemy.
- If you need raw SQL for performance (complex queries), that's fine -- the schema definition should still live in one place.
- Discovered during RepoTracker code review.
