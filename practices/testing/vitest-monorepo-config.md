---
concern: testing
tech: [vitest, typescript, turborepo]
priority: recommended
source-repo: 60k-mono
applies-to: [typescript, vitest]
---
# Vitest Configuration for Monorepos

## PATTERN
Use Vitest as the test runner across a Turborepo monorepo with per-package test configs that extend a shared base. Each package has its own `vitest.config.ts` with package-specific paths and aliases, while Turborepo orchestrates running tests across all packages via the `test` task.

## WHY
Vitest is native to the Vite ecosystem, provides hot module reloading in watch mode, has first-class TypeScript support without compilation, and runs significantly faster than Jest for TypeScript projects. Per-package configs keep tests isolated while Turborepo handles parallelism and caching.

## EXAMPLE
Per-package `vitest.config.ts`:
```typescript
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'node', // or 'jsdom' for UI packages
    include: ['src/**/*.test.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json-summary'],
    },
  },
});
```

Turborepo task (`turbo.json`):
```json
{
  "tasks": {
    "test": {
      "dependsOn": ["^build"]
    }
  }
}
```

Package scripts:
```json
{
  "scripts": {
    "test": "vitest run",
    "test:watch": "vitest",
    "test:coverage": "vitest run --coverage"
  }
}
```

## CHECK
How to verify if a repo already follows this:
- [ ] `vitest` is in devDependencies (root or per-package)
- [ ] `vitest.config.ts` exists in packages that have tests
- [ ] `turbo.json` includes a `test` task
- [ ] Test files follow `*.test.ts` or `*.spec.ts` naming convention

## IMPLEMENT
1. Install vitest: `pnpm add -Dw vitest @vitest/coverage-v8`
2. Create `vitest.config.ts` in each package with tests
3. Add test scripts to each package's `package.json`
4. Add `test` task to `turbo.json`
5. Run initial test suite: `pnpm turbo test`

## NOTES
- Use `environment: 'node'` for backend/API packages, `environment: 'jsdom'` for UI
- Per-package contract tests verify shared package APIs
- Integration tests for behavior, unit tests for logic
- Write tests before implementation when practical
