---
concern: testing
tech: [playwright, typescript]
priority: optional
source-repo: 60k-mono
applies-to: [typescript, playwright]
---
# Playwright E2E Testing

## PATTERN
Use Playwright for end-to-end testing of web applications. Configure with `playwright.config.ts` at the repo root, define test projects for different browsers/viewports, and use the `webServer` option to auto-start the dev server before tests.

## WHY
E2E tests catch integration issues that unit tests miss -- broken routing, auth flows, form submissions, and rendering bugs across browsers. Playwright provides cross-browser testing (Chromium, Firefox, WebKit), auto-waiting, and trace recording for debugging flaky tests.

## EXAMPLE
```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'mobile', use: { ...devices['Pixel 5'] } },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

## CHECK
How to verify if a repo already follows this:
- [ ] `@playwright/test` is in devDependencies
- [ ] `playwright.config.ts` exists at repo root
- [ ] E2E test directory exists (`e2e/` or `tests/`)
- [ ] `webServer` option is configured for auto-start

## IMPLEMENT
1. Install Playwright: `pnpm add -D @playwright/test && npx playwright install`
2. Create `playwright.config.ts` with project configuration
3. Create `e2e/` directory with initial test files
4. Add scripts: `"e2e": "playwright test"`, `"e2e:ui": "playwright test --ui"`
5. Add `test-results/`, `playwright-report/` to `.gitignore`

## NOTES
- Use `webServer.reuseExistingServer` for faster local iteration
- Increase `retries` in CI to handle transient failures
- Use `trace: 'on-first-retry'` to debug flaky tests without overhead on passing tests
- Page Object Model is recommended for larger test suites
