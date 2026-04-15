---
concern: design-systems
tech: [React, Vue, Svelte, HTML, CSS, TypeScript, JavaScript]
priority: foundational
source-repo: BoardPandas/claude-code-bootstrap
applies-to: [React, Vue, Svelte, Angular, HTML, CSS, any-frontend]
---
# Laws of UX Reference for Code Review

## PATTERN

Maintain a structured UX laws reference document (`.claude/references/ux-laws.md`) that maps 30 established UX principles to concrete code-level indicators. Use a dedicated UX review skill and agent that evaluates frontend code against these laws across six dimensions:

1. **Cognitive Load** (Miller's Law, Hick's Law, Choice Overload, Cognitive Load)
2. **Interaction Design** (Fitts's Law, Doherty Threshold, Parkinson's Law, Flow)
3. **Visual Hierarchy and Grouping** (Proximity, Similarity, Common Region, Uniform Connectedness, Pragnanz, Chunking)
4. **User Expectations** (Jakob's Law, Postel's Law, Mental Model, Paradox of Active User)
5. **Memory and Attention** (Miller's Law, Serial Position Effect, Von Restorff Effect, Zeigarnik Effect, Selective Attention, Working Memory)
6. **Experience Quality** (Aesthetic-Usability Effect, Peak-End Rule, Goal-Gradient Effect, Tesler's Law)

Each law entry includes:
- The principle statement
- Key takeaways for practitioners
- Origin and research citation
- "What to look for in code" section with specific patterns to flag

The review produces severity-ranked findings (CRITICAL, WARNING, SUGGESTION), each citing the specific UX law violated, the user impact, and a concrete fix recommendation.

## WHY

- **Consistency:** Without a codified reference, UX reviews depend on individual reviewer knowledge, producing inconsistent feedback across sessions.
- **Actionability:** Generic UX advice ("make it simpler") is hard to act on. Mapping laws to code patterns (e.g., "form has 12 ungrouped fields, violates Miller's Law and Chunking") gives developers specific things to fix.
- **Coverage:** 30 laws organized by category ensures reviews don't fixate on one dimension (e.g., only visual design) while ignoring others (e.g., cognitive load, interaction timing).
- **Teachability:** Citing the law name and origin in each finding educates the team over time, building shared UX vocabulary.
- **Source:** All 30 laws sourced from https://lawsofux.com/ (Jon Yablonski), a widely recognized UX reference.

## EXAMPLE

Reference file structure (`.claude/references/ux-laws.md`):

```markdown
## Decision Making

### 1. Hick's Law
**Principle:** Decision time increases with the number and complexity of choices.
- Minimize choices when response speed matters
- Break complex tasks into manageable steps
- Origin: Hick & Hyman, 1952

**What to look for in code:** Navigation menus with excessive items,
forms presenting all fields at once, settings pages without grouping.
```

UX Review skill (`.claude/skills/ux-review/SKILL.md`):

```yaml
---
name: ux-review
description: Review UI/UX code against Laws of UX and Gestalt principles.
agent: ux-reviewer
allowed-tools: [Read, Glob, Grep, Task]
---
```

Review output format:

```
[WARNING] src/components/SettingsPage.tsx:45 -- Settings page displays 18 options in flat list
  UX Law: Miller's Law + Chunking
  Impact: Users cannot scan or find settings efficiently
  Recommendation: Group into 4-5 logical sections with headers
```

## CHECK

How to verify if a repo already follows this:
- [ ] `.claude/references/ux-laws.md` exists with all 30 laws mapped to code indicators
- [ ] `.claude/agents/ux-reviewer.md` exists with six review dimensions defined
- [ ] `.claude/skills/ux-review/SKILL.md` exists with step-by-step review process
- [ ] CLAUDE.md lists `ux-review` in the Available Skills table
- [ ] `agents.md` lists `ux-reviewer` in the agent registry

## IMPLEMENT

Steps to adopt this in a repo that doesn't have it:

1. Copy `.claude/references/ux-laws.md` from the bootstrap template (contains all 30 laws with code-level indicators organized by category)
2. Copy `.claude/agents/ux-reviewer.md` from the bootstrap template
3. Create `.claude/skills/ux-review/` directory and copy `SKILL.md` from the bootstrap template
4. Add `ux-review` to the Available Skills table in `CLAUDE.md`
5. Add `ux-reviewer` to the Available Agents section in `CLAUDE.md` and to `agents.md`
6. Run `/ux-review` on a sample component to verify the skill works end-to-end

## NOTES

- The 30 laws are sourced from https://lawsofux.com/ and span six categories: decision making, memory/attention, perception/grouping (Gestalt), interaction/performance, user behavior/expectations, and experience design.
- The reference doc should be updated when lawsofux.com adds new laws or when team experience reveals additional code-level indicators.
- The review is most valuable on page-level components, forms, navigation, and multi-step flows. Running it on utility functions or API routes produces no useful findings.
- Pairs well with the design-guardrails reference for visual consistency rules, and with the performance-review skill for Doherty Threshold (400ms response time) validation.
