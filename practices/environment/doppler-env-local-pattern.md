---
concern: environment
tech: [doppler, node]
priority: recommended
source-repo: bp-website
applies-to: [node, typescript]
---
# Doppler + .env.local Fallback Pattern

## PATTERN
Use Doppler as the primary secrets source with `.env.local` as the offline fallback. Development workflow: `doppler run -- npm run dev` injects secrets from Doppler's `dev` config. When offline, fall back to `.env.local` (git-ignored). Document all environment variables in CLAUDE.md with their type (secret vs public) and purpose.

## WHY
Developers need secrets available locally without checking them into git. Doppler centralizes secret management across the team and environments. The `.env.local` fallback ensures development works offline. Documenting variables in CLAUDE.md means Claude Code knows what's available without reading .env files.

## EXAMPLE
From bp-website (`CLAUDE.md`):
```markdown
| Variable | Type | Purpose |
|----------|------|---------|
| `RESEND_API_KEY` | secret | Email delivery (Resend) |
| `SLACK_WEBHOOK_URL` | secret | Contact form notifications |
| `NEXT_PUBLIC_GTM_ID` | public | Google Tag Manager container ID |
| `NEXT_PUBLIC_SITE_URL` | public | Canonical site URL |
```

Development:
```bash
# Primary: Doppler injection
doppler run -- npm run dev

# Fallback: .env.local (offline)
npm run dev  # reads .env.local automatically
```

`.gitignore`:
```
.env
.env.*
!.env.example
```

## CHECK
How to verify if a repo already follows this:
- [ ] `.env.local` is in `.gitignore`
- [ ] CLAUDE.md documents all environment variables
- [ ] No `.env` files with real secrets are committed
- [ ] `doppler run` or equivalent is the primary dev workflow
- [ ] `.env.example` or equivalent documents required variables (without values)

## IMPLEMENT
1. Add `.env*` patterns to `.gitignore` (keep `.env.example`)
2. Create `.env.example` listing all required variables with placeholder values
3. Document all variables in CLAUDE.md with type and purpose
4. Set up Doppler project if not already configured
5. Update dev script documentation to show `doppler run` usage

## NOTES
- Public variables (NEXT_PUBLIC_*) can be committed in `.env.example` with real values
- Secret variables should only have placeholder values in `.env.example`
- Framework-specific: Next.js loads `.env.local` automatically; other frameworks may need `dotenv`
