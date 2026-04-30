---
concern: safety
tech: [typescript, drizzle, multi-tenant]
priority: recommended
source-repo: vigilis
applies-to: [multi-tenant-saas]
---
# tenant-scope-ok Escape Hatch for Multi-Tenant Lint Checks

## PATTERN

Multi-tenant codebases that enforce `WHERE org_id = ?` (or `partner_id = ?`) on every query via a CI script need a structured escape hatch for the small number of routes where the scoping is implicit but correct. Use a single, grep-able comment token (`// tenant-scope-ok`) that the check script treats as a per-file or per-block whitelist, and require the comment to be paired with a sentence explaining WHY the route is safe.

The comment must be:
1. Co-located with the code it justifies (top of the route handler or above the suspect query).
2. Specific enough that a reviewer can re-evaluate it later (cite the column that pins the tenant, the guard in use, or the trust boundary).
3. Discoverable: a single `git grep` for `tenant-scope-ok` returns every concession in the codebase.

## WHY

Multi-tenant isolation is the single highest-stakes invariant in a SaaS codebase: a missed `WHERE` clause leaks data across paying customers. A grep-based check (e.g. "every Drizzle query against a tenant-scoped table includes the tenant column") is cheap, fast, and catches regressions in PRs. But blanket-strict checks always have legitimate exceptions:

- Pre-login routes that look up by domain to render the right SSO buttons.
- Webhook endpoints whose trust comes from a signed payload, not a session.
- Platform-admin routes whose access is gated by support-grant tooling rather than a session orgId.
- Composite indexes where filtering by the parent tenant column makes the child filter implicit.

Without an escape hatch, engineers either disable the check globally (losing the signal) or stop adding new routes (losing velocity). With an escape hatch that requires a justification comment, the false positive becomes an audit artifact: every concession is grep-able, every concession is documented, and security reviews can re-walk the list.

The pattern also acts as a tripwire during code review: if a PR adds a new `tenant-scope-ok` comment, the reviewer knows to read the justification before approving.

## EXAMPLE

From vigilis (`src/app/api/auth-methods/route.ts`):

```typescript
// tenant-scope-ok: this is an unauthenticated pre-login endpoint that maps an
// email domain to its company's auth-method flags so the login form can render
// the right buttons. The lookup keys on companies.domain (a public identifier);
// the join to orgSettings.orgId pulls only the auth-method flag columns.
// No tenant data is returned -- the response is just a 4-boolean flag set.

export async function GET(req: NextRequest) {
  const email = req.nextUrl.searchParams.get("email");
  // ... domain lookup, returns only auth-method booleans
}
```

From the same repo (`src/app/api/org-settings/auth/route.ts`):

```typescript
// tenant-scope-ok: this route allows platform_admin to read/write a target
// org's auth settings via a `?orgId=` query param, bypassing the session's
// active org. All queries filter by `targetOrgId` and organization.partnerId
// is NOT NULL, so the partner is implicit in that single-org filter. The
// platform_admin override path is currently unguarded by an active support
// grant -- this is the security review Phase 7 L1 finding, tracked in
// docs/compliance/SOC2-CONTROLS.md open-items table.
```

The check script (`scripts/check-tenant-scope.sh`) skips files whose flagged blocks carry the comment, but reports the file in a "concessions" tally so the count is visible in CI output.

## CHECK

How to verify if a repo follows this pattern:
- [ ] A CI/lint script runs on every PR enforcing tenant scoping (e.g. `pnpm check:tenant-scope`).
- [ ] The script recognises a single, documented escape-hatch token (`// tenant-scope-ok`, `# tenant-scope-ok`, etc.).
- [ ] Every occurrence of the token is paired with a justification comment in the same file.
- [ ] `git grep tenant-scope-ok` returns a small, finite, audit-able list (typically less than 1% of routes).
- [ ] PR review checklist mentions: "if this PR adds a tenant-scope-ok, read the justification".

## IMPLEMENT

1. Add a check script that greps tenant-scoped tables for queries missing the tenant column.
2. Make the script recognize `tenant-scope-ok` as a per-file or per-block opt-out.
3. Require the comment to be followed by free-text justification on the same comment block (the script can enforce length/presence).
4. Wire the script into CI (`pnpm check:tenant-scope` runs on every PR).
5. Add a doc entry (e.g. `.claude/rules/api.md`) explaining when it is OK to add the comment versus when the route should be rewritten.
6. During quarterly security review, re-walk every `tenant-scope-ok` site and confirm the justification still holds.

## NOTES

- Pair this with structured guards (`withAuth`, `withPartnerScope`, etc.) so the escape hatch is rarely needed.
- The justification comment is the audit artifact, not the check itself -- if the comment says "TODO: justify later", reject the PR.
- For SOC 2 / ISO 27001 audits, the list of `tenant-scope-ok` sites maps cleanly to a "exceptions register" control.
- Avoid blanket file-level disables (`/* tenant-scope-ok-file */`); per-block comments keep the surface area small.
