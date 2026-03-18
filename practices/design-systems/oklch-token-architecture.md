---
concern: design-systems
tech: [css, oklch, tailwind]
priority: optional
source-repo: wellforce-design-system
applies-to: [css, tailwind, design-systems]
---
# OKLCH Token Architecture

## PATTERN
Use OKLCH color space for all design tokens. Store colors as raw OKLCH triplets (`0.65 0.2 260`) without the `oklch()` wrapper, compose at usage time in CSS (`oklch(var(--primary))`). Derive 55 semantic color roles from 9 seed colors using OKLCH math (mix, lighten, darken, hue rotate). Every background token has a matching foreground token with WCAG AA contrast guarantee.

Three-tier system:
1. **Primitive tokens** (rare): `--blue-500` raw palette values
2. **Semantic tokens** (primary): `--primary: 0.65 0.2 260` named roles
3. **Composed at usage**: `background: oklch(var(--primary))`

## WHY
OKLCH is perceptually uniform -- the Lightness (L) channel directly controls perceived brightness, making it trivial to auto-derive accessible color pairs. Traditional hex/HSL can't guarantee contrast from math alone. Storing raw triplets (not wrapped in `oklch()`) enables composability -- you can manipulate individual channels in CSS calc() or derive new colors without parsing.

## EXAMPLE
From wellforce-design-system (`CLAUDE.md` guardrails):
```markdown
Rule 2: OKLCH color space internally. Store all colors as OKLCH.
         Convert to hex/HSL only at export time.

Rule 5: Raw values without wrapper. Store as --primary: 0.65 0.2 260
         not oklch(0.65 0.2 260). Compose in usage: oklch(var(--primary))

Rule 4: Background/foreground pairs. Every background MUST have matching
         -foreground token for guaranteed contrast.
```

Color derivation from 9 seeds to 55 roles:
```typescript
// Input: 9 seed colors
interface SeedColors {
  background, foreground, primary, secondary,
  accent, destructive, success, warning, sidebar
}

// OKLCH math operations
const card = lighten(bg, 0.02);        // Slight elevation
const muted = mix(bg, fg, 0.10);      // Subtle text
const border = mix(bg, fg, 0.15);     // Visible boundary
const green = hueRotate(primary, 120); // Complementary hue

// Output: 55 semantic roles (hex strings)
// Core, Identity, Functional, Status, Palette, Sidebar, UI
```

CSS usage:
```css
:root {
  --primary: 0.65 0.2 260;
  --primary-foreground: 0.98 0.005 260;
}

.button-primary {
  background: oklch(var(--primary));
  color: oklch(var(--primary-foreground));
}
```

## CHECK
How to verify if a repo already follows this:
- [ ] Colors stored as OKLCH values (not hex/HSL) in token definitions
- [ ] Raw triplets stored without `oklch()` wrapper
- [ ] Background tokens have matching `-foreground` pairs
- [ ] WCAG AA contrast verified in the token layer

## IMPLEMENT
1. Install culori for OKLCH conversions: `npm install culori`
2. Convert existing hex/HSL tokens to OKLCH triplets
3. Create semantic token map with background/foreground pairs
4. Build derivation functions (mix, lighten, darken, hueRotate) using OKLCH math
5. Set up export pipeline: OKLCH -> hex, HSL, Tailwind, shadcn, CSS vars

## NOTES
- 22 mandatory guardrails in wellforce-design-system cover tokens, rendering, accessibility, and architecture
- Max 2 levels of variable indirection -- no deep `--a -> --b -> --c -> --d` chains
- Follow Open Props naming: `--surface-1`, `--text-1`, not custom namespaces
- Iframe isolation for template previews prevents cross-boundary style leakage
- Export pipeline covers: CSS Variables, Tailwind, shadcn/ui, Bootstrap, Vuetify, Material UI, Chakra UI, DTCG JSON
