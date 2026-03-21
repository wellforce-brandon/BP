---
concern: deployment
tech: [nextjs, cloudflare, vercel, deployment]
priority: recommended
source-repo: wellforce-design-system
applies-to: [nextjs, cloudflare, vercel]
---
# Build version console logging for deployment verification

## PATTERN
Log the application version (from `package.json`) in the root layout or app entry point so you can verify which deployment is live by checking server logs. This is critical when debugging production issues where you need to confirm your latest code has actually deployed.

## WHY
When debugging production issues, the first question is always "is the latest code actually deployed?" Without a version log, you have to guess based on behavior or dig through deployment dashboards. A simple console.log at startup eliminates this uncertainty immediately. This is especially important with Cloudflare Workers where deployments can take a few seconds to propagate, and with edge caching where stale responses may persist.

## EXAMPLE
```typescript
// app/layout.tsx (Next.js root layout)
import packageJson from "../package.json";

console.log(`[app] v${packageJson.version} loaded`);

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return <html><body>{children}</body></html>;
}
```

Server logs will show:
```
[app] v1.7.0.3 loaded
```

## CHECK
How to verify if a repo already follows this:
- [ ] Root layout or app entry point logs the version from package.json
- [ ] Version is visible in server logs (Cloudflare, Vercel, etc.)

## IMPLEMENT
1. Import `package.json` in the root layout (or entry point)
2. Add `console.log(\`[app] v${packageJson.version} loaded\`)`
3. Verify the log appears in your deployment platform's log viewer

## NOTES
- Keep it to a single line -- this is for quick verification, not verbose startup logging.
- For Cloudflare Workers, logs are visible in `wrangler tail` or the Cloudflare dashboard under Workers > Logs.
- Consider also logging the build timestamp if your build pipeline supports it.
