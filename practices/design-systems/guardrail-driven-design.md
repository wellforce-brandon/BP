---
concern: design-systems
tech: [css, tailwind, react]
priority: recommended
source-repo: 60k-mono
applies-to: [typescript, react, tailwind]
---
# Guardrail-Driven Design System

## PATTERN
Define a comprehensive design guardrails document (`.claude/references/design-guardrails.md`) that constrains all UI decisions: component size limits, spacing scales, color token usage, accessibility requirements, and performance budgets. Claude Code reads this before any UI work, preventing inconsistency without requiring a full component library.

## WHY
Without guardrails, AI-generated UI drifts -- different spacing, inconsistent colors, missing accessibility, and bloated bundles. A guardrails document acts as a contract: Claude can freely build UI within the constraints but cannot violate them. This is more effective than verbal instructions because it's loaded automatically via the reference files pattern.

## EXAMPLE
From 60k-mono (`.claude/references/design-guardrails.md`):
```markdown
## Component Rules
- Max 200 lines per file, max 8 props per component
- Composition via children and render props, not prop drilling
- Boolean props: is* or has* prefix
- Use cva for multi-variant components

## Styling
- CSS-first config (@theme in CSS, not tailwind.config.js)
- Semantic color tokens only (bg-primary, not bg-blue-500)
- Mobile-first responsive (sm:, md:, lg:, xl:)
- No @apply, no inline style={} except dynamic values
- Dark mode via CSS class-based strategy on <html>

## Accessibility (WCAG AA)
- 4.5:1 contrast for normal text
- All interactive elements keyboard accessible
- Focus indicators: visible ring, never outline-none without replacement
- Skip-to-content link on every page

## Performance
- Initial JS: < 150 kB gzipped
- LCP: < 2.5s, FID: < 100ms, CLS: < 0.1
- React Router lazy routes for code splitting
- TanStack Virtual for >500 row lists
```

## CHECK
How to verify if a repo already follows this:
- [ ] `.claude/references/design-guardrails.md` exists
- [ ] Guardrails document covers: components, styling, accessibility, performance
- [ ] CLAUDE.md references the guardrails file in its documentation table
- [ ] UI code generally adheres to the documented constraints

## IMPLEMENT
1. Create `.claude/references/design-guardrails.md`
2. Define constraints for: component rules, styling approach, spacing scale, color system, accessibility requirements, performance budgets, typography
3. Link from CLAUDE.md documentation table with "Read Before: any UI work"
4. Customize constraints to match your stack (Chakra vs shadcn vs Tailwind-only, etc.)

## NOTES
- The guardrails document is a reference file, not a CLAUDE.md section -- keeps CLAUDE.md lean
- Guardrails should be specific to the project's stack, not generic CSS advice
- Include concrete numbers (max props, max lines, performance budgets) for mechanical enforcement
- Update guardrails when the design system evolves
