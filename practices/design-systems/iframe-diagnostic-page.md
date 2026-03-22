---
concern: design-systems
tech: [react, nextjs, iframe, css]
priority: recommended
source-repo: wellforce-design-system
applies-to: [design-systems, iframe, debugging]
---
# Iframe Rendering Diagnostic Page

## PATTERN
When debugging rendering issues inside iframes (fonts, styles, scripts), create a self-contained diagnostic page that isolates each variable independently. Each test case should be a minimal reproduction that tests exactly one thing: network fetch, encoding, CSS parsing, iframe context, or visual rendering. Show pass/fail results visually on the page itself.

## WHY
Iframe rendering bugs are notoriously hard to diagnose because the pipeline has many stages (asset loading, CSS injection, document context, CORS, blob URLs). When the full pipeline fails, you can't tell which stage is broken. A diagnostic page that tests each stage independently narrows the root cause in minutes instead of days.

In the BoardPandas Design Studio, this pattern resolved a 15-attempt, 2-day font rendering investigation by proving that the WOFF2 files themselves were the issue -- not CORS, not iframes, not base64 encoding, not the CSS pipeline.

## EXAMPLE
```typescript
// app/debug/font-test/page.tsx -- self-contained diagnostic
export default function FontTestPage() {
  return <FontTestClient />;
}

// client.tsx -- tests each variable independently
function FontTestClient() {
  const [results, setResults] = useState<TestResult[]>([]);

  useEffect(() => {
    runTests().then(setResults);
  }, []);

  return (
    <div>
      <h1>Font Rendering Diagnostics</h1>
      {results.map((r) => (
        <div key={r.name}>
          <span>{r.pass ? 'PASS' : 'FAIL'}</span> {r.name}: {r.detail}
        </div>
      ))}
      {/* Minimal iframe with ONE font, no pipeline dependencies */}
      <iframe srcDoc={minimalFontTestHtml} />
    </div>
  );
}

async function runTests(): Promise<TestResult[]> {
  return [
    await testFetch('/fonts/example-400.woff2'),     // Can we fetch?
    await testMagicBytes('/fonts/example-400.woff2'), // Valid WOFF2?
    await testBase64Roundtrip(fontData),              // Encoding OK?
    // Visual test: iframe renders font vs fallback side by side
  ];
}
```

## CHECK
How to verify if a repo already follows this:
- [ ] When iframe rendering bugs occur, is there a dedicated diagnostic page rather than debugging through the full pipeline?
- [ ] Does the diagnostic page test each variable (network, encoding, rendering) independently?

## IMPLEMENT
Steps to adopt this:
1. Create a route like `/debug/<feature>-test` (excluded from production builds if needed)
2. Build one test per pipeline stage, each returning pass/fail with details
3. Include a minimal iframe that tests the rendering in isolation (no shared CSS, no pipeline code)
4. Show results visually on the page -- no need to check console

## NOTES
- Delete the diagnostic page after the issue is resolved, or gate it behind a dev-only route.
- The key insight is testing each variable independently. A test that uses the full pipeline tells you "it's broken" but not where.
- Auto-discovered from wellforce-design-system font-loading investigation (15 attempts).
