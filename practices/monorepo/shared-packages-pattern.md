---
concern: monorepo
tech: [pnpm, typescript]
priority: recommended
source-repo: 60k-mono
applies-to: [typescript, monorepo]
---
# Shared Packages with Barrel Exports and Layer Dependencies

## PATTERN
Each shared package in `packages/` exports from a barrel file (`src/index.ts`), uses a common `@namespace/` prefix, and respects a numbered layer hierarchy that enforces import direction. Higher-layer packages can import from lower layers but never the reverse.

## WHY
Without explicit dependency layers, monorepos devolve into circular dependencies and spaghetti imports. Barrel exports provide a stable public API for each package, and layer numbering makes the dependency graph immediately visible to both humans and AI assistants.

## EXAMPLE
From 60k-mono (`.claude/agent-memory/patterns.md`):
```markdown
## Package Structure
- All shared packages: @devtools/ namespace with workspace:* versions
- Barrel exports from src/index.ts
- Layer dependency:
  0 (config/utils) -> 1 (database) -> 2 (auth/billing/api-framework)
  -> 3 (notifications/mcp-server/ui) -> 4 (apps/workers)
```

Each package's `package.json`:
```json
{
  "name": "@devtools/database",
  "main": "src/index.ts",
  "types": "src/index.ts",
  "exports": {
    ".": "./src/index.ts"
  }
}
```

Layer enforcement in path-scoped rules (`.claude/rules/workers.md`):
```markdown
- Never import from `apps/`, only from `packages/`
- Respect dependency layer (workers = Layer 4, import from 0-3)
```

## CHECK
How to verify if a repo already follows this:
- [ ] All shared packages have a `src/index.ts` barrel export
- [ ] Packages use a common `@namespace/` prefix
- [ ] Import directions are documented (dependency layers)
- [ ] No circular dependencies between packages
- [ ] Workers/apps don't import from each other, only from packages

## IMPLEMENT
1. Create each shared package with `src/index.ts` exporting its public API
2. Use `@namespace/package-name` naming convention for all packages
3. Document the layer hierarchy in CLAUDE.md or agent-memory
4. Add path-scoped rules enforcing import direction
5. Verify with: `pnpm turbo build` (topological build will fail on circular deps)

## NOTES
- TypeScript strict mode across all packages; no `any` in shared exports
- Database schemas: one schema file per domain entity, all with `withTimezone: true`
- API routes: tRPC routers in the API app, REST only for webhooks and health checks
- Each package should be independently testable with its own test config
