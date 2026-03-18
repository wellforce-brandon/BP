---
concern: monorepo
tech: [pnpm, turborepo, typescript]
priority: recommended
source-repo: 60k-mono
applies-to: [typescript, monorepo]
---
# pnpm + Turborepo Monorepo Structure

## PATTERN
Use pnpm workspaces for package management and Turborepo for task orchestration. Organize code into `packages/` (shared libraries), `apps/` (deployable applications), and `workers/` (background processors). All shared packages use a common namespace (e.g., `@devtools/`) with `workspace:*` version references.

## WHY
Monorepos enable code sharing without publishing to npm, consistent tooling across projects, and atomic cross-package changes. pnpm's strict dependency resolution prevents phantom dependencies. Turborepo's task graph caches builds and runs tasks in topological order, dramatically speeding up CI.

## EXAMPLE
From 60k-mono:

`pnpm-workspace.yaml`:
```yaml
packages:
  - "packages/*"
  - "apps/*"
  - "workers/*"
  - "extensions/*"
```

`turbo.json`:
```json
{
  "$schema": "https://turbo.build/schema.json",
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "lint": {},
    "test": {}
  }
}
```

Directory structure:
```
packages/    auth, billing, notifications, api-framework, mcp-server, ui, database, scan-engine
apps/        windrunner, logpulse, marketing, api, admin
workers/     scan-worker, notification-sender, uptime-checker, cron-evaluator, visual-scanner
```

Package references use `workspace:*`:
```json
{
  "dependencies": {
    "@devtools/database": "workspace:*",
    "@devtools/auth": "workspace:*"
  }
}
```

## CHECK
How to verify if a repo already follows this:
- [ ] `pnpm-workspace.yaml` exists at repo root
- [ ] `turbo.json` exists with task definitions
- [ ] Packages use `workspace:*` version references
- [ ] Shared code lives in `packages/` with a common namespace
- [ ] Build tasks use `dependsOn: ["^build"]` for topological ordering

## IMPLEMENT
1. Install pnpm and Turborepo: `pnpm add -Dw turbo`
2. Create `pnpm-workspace.yaml` listing package directories
3. Create `turbo.json` with build, dev, lint, test task definitions
4. Move shared code to `packages/<name>/` with its own `package.json`
5. Reference shared packages via `"@namespace/name": "workspace:*"`
6. Add barrel exports (`src/index.ts`) to each shared package

## NOTES
- Layer dependencies enforce import direction: packages -> apps -> workers (never reverse)
- Example layers: 0 (config/utils) -> 1 (database) -> 2 (auth/billing) -> 3 (notifications/ui) -> 4 (apps/workers)
- Workers should never import from `apps/`, only from `packages/`
- Use `pnpm --filter <package> add <dep>` to add deps to specific packages
