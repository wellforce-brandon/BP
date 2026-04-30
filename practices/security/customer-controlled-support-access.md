---
concern: security
tech: [saas, multi-tenant, better-auth, drizzle, postgres, soc2, gdpr]
priority: foundational
source-repo: vigilis
applies-to: [b2b-saas, multi-tenant-platforms]
---
# Customer-Controlled Support Access for Multi-Tenant SaaS

## PATTERN
Platform staff have **zero default access** to tenant data. Tenant admins issue time-bound, scope-bounded support access grants that are visible, revocable, and audit-logged. Three workflows cover all legitimate access needs:

1. **Standard grant** -- tenant admin explicitly grants access (scope, duration, reason). Sensible default: 24h read-only single-org. Hard cap 30d. Visible banner during use. Tenant can revoke at any time.

2. **Break-glass** -- emergency access for incidents. Requires co-sign from a second platform admin. 4h hard expiry. Tenant notified on activation via email + in-app + Slack. Telemetry alert fires.

3. **Legal Hold (concealed)** -- for legal investigations only. Requires a designated Legal Officer co-sign in addition to the standard break-glass co-sign. Concealed from tenant during investigation. Auto-discloses retroactively when hold releases.

Schema (Drizzle/Postgres):
- `support_access_grants` -- grant records with scope enum, expiry, revocation, concealment_mode
- `support_access_audit` (append-only) -- every action under a grant, with grant_id reference
- Mirror Legal Hold writes to immutable cold storage (object-lock S3 / Railway Storage write-once retention) for tamper evidence

UI:
- Tenant admin: grant form, active grant card with countdown + revoke, full audit history
- Platform admin: request-access flow, persistent banner during support mode, exit button
- Auto-expiry sweep job (BullMQ hourly)
- Quarterly auto-emailed PDF audit report to tenant admins

## WHY
- **SOC 2 alignment**: maps to CC6.1 (logical access), CC6.6 (access boundaries), CC7.3 (incident detection).
- **GDPR data-processor model**: tenant is controller, platform is processor. Access requires explicit controller authorization.
- **Limits credential-compromise blast radius**: a stolen platform admin token cannot read tenant data without a grant.
- **Sales-positive**: "Vendor staff cannot see your data without your permission" closes deals with security-conscious customers.
- **Audit trail for legal/regulatory**: subpoenas, internal investigations, compliance requests have an immutable record.
- **Industry-proven**: GitHub, Stripe, Auth0, Linear, Intercom, 1Password all implement variants of this pattern.

## EXAMPLE
From `tasks/multi-broker-architecture.md` (vigilis):

```ts
// support_access_grants schema
{
  id, partnerId, orgId,
  scope: enum('broker_metadata' | 'single_org_read' | 'single_org_write' | 'full_broker_read' | 'full_broker_write'),
  grantedByUserId, grantedToUserId,
  reason: text,
  isBreakGlass: bool, breakGlassCoSignerId,
  isLegalHold: bool, legalHoldCaseId, legalOfficerCoSignerId,
  concealmentMode: enum('visible' | 'legal_hold'),
  createdAt, expiresAt, revokedAt, revokedByUserId,
  firstUsedAt, lastUsedAt, useCount,
  holdReleasedAt
}

// Guard chain
if (actor.role === 'platform_admin' && resource.partnerId !== null) {
  const grant = activeGrantFor(actor, resource.partnerId, resource.orgId, requiredScope)
  if (!grant) return 403
  await logSupportAccessUse(grant, actor, resource, action)
}
```

## CHECK
How to verify if a repo already follows this:
- [ ] Platform staff cannot read tenant data by default (verified by integration test that asserts 403 without active grant)
- [ ] Grant table records every access event with grant_id, actor, resource, timestamp
- [ ] Grants auto-expire via server-side sweep job (not client-side or middleware-only)
- [ ] Revocation takes effect mid-session (no in-flight requests honored after revoke)
- [ ] Persistent UI banner during support mode, not dismissible by platform admin
- [ ] Break-glass requires second-platform-admin co-sign before activation
- [ ] Concealed/Legal Hold path requires distinct Legal Officer co-sign and writes to immutable storage
- [ ] Tenant receives quarterly PDF audit report
- [ ] Audit retention policy meets compliance requirements (recommend 7 years for SOC 2)

## IMPLEMENT
Steps to adopt this in a repo that does not have it:
1. Add `support_access_grants` and `support_access_audit` schema (Drizzle migration).
2. Add a `withSupportAccessOr(roleCheck)` middleware that checks for active grant before allowing platform admin through tenant-scoped guards.
3. Sweep through all tenant-scoped API routes; replace any "platform admin bypasses tenant filter" logic with the grant-aware middleware.
4. Build tenant-side grant management UI (issue, revoke, audit view).
5. Build platform-side request flow + persistent support-mode banner that cannot be dismissed.
6. Add BullMQ hourly job for grant expiry + telemetry events.
7. Add break-glass workflow (separate route, 2-admin co-sign, alerts on activation).
8. Add Legal Hold workflow (Legal Officer flag/role, concealment with auto-disclosure on hold release).
9. Build quarterly PDF audit cron job.
10. Document in security runbook; verify against SOC 2 control map.

## NOTES
- **Default duration tradeoff**: 24h spans an overnight; some shops prefer 4h default with 24h as a one-click preset. Pick based on tenant expectations.
- **Concealed grants must still write to immutable storage simultaneously with DB**: DB-only writes can be tampered with by sufficiently privileged actors. Use object-lock S3 (or Railway Storage write-once retention).
- **Legal Hold auto-disclosure is non-negotiable** for trust: concealment is *temporary*, not permanent. When the hold releases, the tenant gets the full audit retroactively.
- **Pin grants to specific staff** for paranoid tenants: optional `granted_to_user_id` so a grant only works for one named platform admin.
- **Read vs. write distinction**: most support is diagnostic. Default scope read-only; write access is a separate, more deliberate grant.
- **Industry analogs to reference when explaining**: GitHub support access toggle, Stripe read-only support PIN, Auth0 temporary access, Linear customer-controlled support, Intercom time-bound access.
- **Onboarding paradox does not exist** if tenant creation always pairs with designating an initial tenant admin in the same transaction. Skip "implicit setup access" designs.
