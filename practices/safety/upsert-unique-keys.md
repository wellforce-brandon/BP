---
concern: safety
tech: [sqlite, typescript, drizzle]
priority: foundational
source-repo: RepoTracker
applies-to: [sqlite, typescript]
---
# Use Unique Identifiers for Database Upserts

## PATTERN
Always use a truly unique identifier (file path, URL, UUID, or compound key) as the lookup key for database upserts. Never use a display name, directory name, or other human-readable label that can collide across contexts.

## WHY
Display names collide. Two parent directories can each contain a subdirectory named "api". Two GitHub orgs can each have a repo named "web". Two users can have a project called "my-app". When the upsert key is the display name, the second record silently overwrites the first, merging two unrelated entities into one. This is a data integrity issue that produces no errors -- the app appears to work while silently losing data.

## EXAMPLE
From RepoTracker (`src/hooks/useRepos.ts`):

Wrong -- matches by name, which is not unique:
```typescript
const existing = await db
  .select()
  .from(repository)
  .where(eq(repository.name, local.name));
// Two repos named "api" collide
```

Right -- matches by unique local path:
```typescript
const existing = await db
  .select()
  .from(repository)
  .where(eq(repository.localPath, local.path));
// C:/Github/org1/api and C:/Github/org2/api are distinct
```

Right -- compound key when no single field is unique:
```typescript
const existing = await db
  .select()
  .from(repository)
  .where(
    and(
      eq(repository.name, local.name),
      eq(repository.localPath, local.path)
    )
  );
```

## CHECK
- [ ] Search for `.where(eq(table.name,` in upsert logic -- likely a collision risk
- [ ] Verify all upsert/merge operations use a column with a UNIQUE constraint
- [ ] Check if the lookup column can contain duplicates across different contexts (orgs, directories, users)

## IMPLEMENT
1. Identify the truly unique field for each entity (path, URL, email, UUID)
2. Add a UNIQUE constraint on that column in the schema
3. Change all upsert/merge queries to match on the unique column
4. Keep the display name as a separate non-unique column for UI rendering

## NOTES
- This applies to any data source: filesystem scans, API imports, CSV ingestion.
- A UNIQUE constraint at the DB level provides a safety net even if application code has bugs.
- For entities that have no natural unique key, generate a UUID at creation time.
- Discovered during RepoTracker code review.
