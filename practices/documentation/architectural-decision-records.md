---
concern: documentation
tech: [claude-code]
priority: recommended
source-repo: external
applies-to: [all]
---
# Architectural Decision Records (ADRs)

## PATTERN
Document significant architectural decisions in a structured format: context (what prompted the decision), decision (what was chosen), alternatives (what was considered), consequences (trade-offs accepted), and status (proposed/accepted/deprecated/superseded). Store in `docs/adr/` or `.claude/agent-memory/decisions.md` for AI visibility.

## WHY
Reference files capture "what the code does." LL-G captures "what went wrong." ADRs capture "why we chose X over Y" -- the missing middle. Without ADRs, teams (and AI assistants) re-evaluate settled decisions, propose alternatives that were already rejected, or unknowingly reverse intentional trade-offs. ADRs make the invisible rationale visible.

## EXAMPLE
From 60k-mono (`.claude/agent-memory/decisions.md`):
```markdown
## Drizzle 0.45.x (not v1 beta)
- v1.0.0-beta has breaking migration structure changes and RQBv2
- Stay on 0.45.x until deliberate upgrade window planned
- Caret range won't auto-resolve to v1 beta (tagged `beta`, not `latest`)

## Single-stage tsx Dockerfiles
- All Northflank services use tsx runtime (no build step in Docker)
- Keeps Dockerfiles simple and consistent across all 6 services

## Redis for caching + queues
- Feature flags: Redis cache with 60s TTL
- Blocklist: Redis SETs with 60s refresh
- Job queues: BullMQ backed by same Redis instance
```

Full ADR format (for formal decisions):
```markdown
# ADR-001: Use Fastify over Express

## Status
Accepted (2026-01-15)

## Context
Need a Node.js HTTP framework for the API server.

## Decision
Fastify 5.x with tRPC plugin.

## Alternatives Considered
- Express 5.x: Slower, less type-safe plugin system
- Hono: Lighter but less mature plugin ecosystem

## Consequences
- Pro: 2x throughput, native JSON schema validation
- Con: Smaller community than Express, some middleware gaps
```

## CHECK
- [ ] `.claude/agent-memory/decisions.md` or `docs/adr/` directory exists
- [ ] At least one architectural decision is documented
- [ ] Decisions include rationale (why), not just the choice (what)

## IMPLEMENT
1. Create `.claude/agent-memory/decisions.md` for AI-visible decisions
2. For formal ADRs, create `docs/adr/` with numbered files (ADR-001, ADR-002)
3. Document existing undocumented decisions retroactively
4. Add to CLAUDE.md: "Major architectural decisions go in decisions.md"

## NOTES
- Source: awesome-copilot create-architectural-decision-record skill, 60k-mono agent-memory
- Lightweight format (in agent-memory) is fine for most decisions
- Full ADR format is for decisions that affect multiple teams or are hard to reverse
- ADRs complement LL-G: LL-G captures failures, ADRs capture intentional choices
