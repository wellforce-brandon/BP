---
concern: error-handling
tech: [typescript, react, tauri]
priority: foundational
source-repo: RepoTracker
applies-to: [typescript, react, tauri]
---
# Promise-Based Singleton for Async Resources

## PATTERN
When lazy-initializing an async resource (database connection, auth session, config), store the initialization **promise**, not the resolved value. All concurrent callers share the same in-flight promise, preventing duplicate resource creation.

## WHY
Any async resource called from multiple React hooks, components, or event handlers simultaneously will race if guarded by a null-check on the resolved value. The first caller sets the value to null, starts `await load()`, and before it resolves, the second caller also sees null and starts a duplicate `load()`. For database connections, this creates multiple pools; for auth tokens, multiple refresh cycles.

## EXAMPLE
From RepoTracker (`src/lib/drizzle-bridge.ts`):
```typescript
// Store the promise, not the instance
let dbPromise: Promise<Database> | null = null;

function getDb(): Promise<Database> {
  if (!dbPromise) {
    dbPromise = Database.load("sqlite:repotracker.db");
  }
  return dbPromise;
}
```

For error recovery (retry on failure):
```typescript
let dbPromise: Promise<Database> | null = null;

function getDb(): Promise<Database> {
  if (!dbPromise) {
    dbPromise = Database.load("sqlite:app.db").catch((err) => {
      dbPromise = null; // allow retry on next call
      throw err;
    });
  }
  return dbPromise;
}
```

## CHECK
- [ ] Search for `let.*=.*null` followed by `async function` that assigns to the same variable
- [ ] Check database, auth, and config initialization code for the check-then-await pattern
- [ ] Verify singleton resources are initialized exactly once in React DevTools Network tab

## IMPLEMENT
1. Change the variable type from `T | null` to `Promise<T> | null`
2. Remove `await` from the assignment -- store the raw promise
3. Change the getter return type from `Promise<T>` to just returning the stored promise
4. If retry-on-failure is needed, add a `.catch()` that resets the promise to null

## NOTES
- This pattern is framework-agnostic -- works in React, Vue, Node.js, Deno, etc.
- Especially critical in Tauri apps where multiple IPC calls hit the singleton simultaneously at startup.
- Discovered during RepoTracker code review.
