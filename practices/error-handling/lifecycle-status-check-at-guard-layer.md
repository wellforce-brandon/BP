---
concern: error-handling
tech: [nextjs, typescript]
priority: recommended
source-repo: vigilis
applies-to: [multi-tenant, saas]
---
# Tenant lifecycle status checks belong at the auth-guard layer, not in each route

## PATTERN
For multi-tenant SaaS where tenants can be in non-active lifecycle states (suspended, locked, archived, frozen-for-billing, deactivated), enforce the write-block at the **auth-guard layer**, not at each individual route. Inject a single helper into your `withOrgAdmin`, `withTenantAdmin`, etc. that:

1. Detects the HTTP method is mutating (POST/PATCH/PUT/DELETE).
2. Resolves the tenant ID from URL params (or by lookup from the session's active org).
3. Loads the tenant's lifecycle status.
4. If status is in a write-blocked set, returns **423 Locked** before the route handler runs.

The route handler stays unchanged. Every existing mutation route across the codebase picks up the enforcement automatically — services, invoices, contacts, comments, settings, etc. — without per-route diffs.

## WHY
- **One source of truth.** Adding a new lifecycle status (say, `read_only_audit_mode`) is a one-line change in the predicate, not 80 changes across routes.
- **Impossible to forget.** Per-route checks rely on every route author remembering the new gate. Guard injection makes it impossible to forget — if the route uses the auth guard (which all mutation routes must), it's enforced.
- **Composable with role checks.** The guard already knows the partner ID for role enforcement; adding the lifecycle check at the same boundary is cheap (one extra cached DB call).
- **HTTP method discrimination.** Reads pass through; only mutations are gated. This matches the actual semantics of "suspended": the tenant can still view their data during the grace period, just can't write.
- **423 is the right status code.** "Locked" semantically matches "this resource exists, you have permission, but the resource itself is in a state that disallows the mutation." Distinct from 403 (you're not allowed) and 409 (request conflicts with current state).
- **Pairs with cascade workers.** A daily cron walks tenants through `active -> past_due -> suspended -> locked -> archived`. The guard layer is what makes those state transitions visible to the API immediately, with no application code changes per state.

## EXAMPLE
The lifecycle predicate (`src/lib/auth/partner-lifecycle.ts` from vigilis):
```ts
const WRITE_BLOCKED_STATUSES = new Set(["suspended", "locked", "archived"]);

export async function getPartnerLifecycle(partnerId: string) {
  const [row] = await db.select(...).from(partners).where(eq(partners.id, partnerId));
  if (!row) return null;
  return {
    id: row.id,
    status: row.status,
    isWriteBlocked: WRITE_BLOCKED_STATUSES.has(row.status),
  };
}
```

The guard helper (`src/lib/api/auth-guard.ts`):
```ts
const MUTATING_METHODS = new Set(["POST", "PATCH", "PUT", "DELETE"]);

async function enforcePartnerWriteOnMutation(
  req: NextRequest,
  partnerId: string | null,
): Promise<NextResponse | null> {
  if (!partnerId) return null;
  if (!MUTATING_METHODS.has(req.method.toUpperCase())) return null;
  const lifecycle = await getPartnerLifecycle(partnerId);
  // Missing partners aren't the guard's concern — let the handler's own
  // existence checks return 404. The guard only enforces lifecycle status.
  if (!lifecycle) return null;
  if (!lifecycle.isWriteBlocked) return null;
  return NextResponse.json(
    { error: `Partner is ${lifecycle.status}; writes are blocked. Contact your provider.` },
    { status: 423 },
  );
}
```

Injected into the role guards:
```ts
export function withOrgAdmin(handler: RouteHandler) {
  return async (req, routeCtx) => {
    const session = await auth.api.getSession({ headers: req.headers });
    if (!session) return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    if (!isOrgAdmin(session.user.role)) return NextResponse.json({ error: "Forbidden" }, { status: 403 });

    const orgId = session.session.activeOrganizationId ?? null;
    if (orgId) {
      const partnerId = await getPartnerIdForOrg(orgId);
      const blocked = await enforcePartnerWriteOnMutation(req, partnerId);
      if (blocked) return blocked;
    }

    return handler(req, { session, orgId, params });
  };
}
```

## CHECK
- [ ] Lifecycle status is stored as a typed enum on the tenant table (not a boolean `isActive`).
- [ ] A central predicate (`isWriteBlockedStatus(status)`) lists every write-blocked state.
- [ ] The predicate is called from inside the auth guard, not from individual route handlers.
- [ ] The check runs only on mutating HTTP methods.
- [ ] Returns 423 Locked with a human-readable reason, not 403.
- [ ] When tenant doesn't exist, the guard passes through (route handler returns 404).
- [ ] Tests exist for both blocked and non-blocked states.

## IMPLEMENT
1. Define a typed enum for tenant lifecycle status (e.g. `["active", "suspended", "locked", "archived"]`).
2. Build a small `getTenantLifecycle(tenantId)` helper that returns `{ status, isWriteBlocked }`.
3. Add `enforceWriteOnMutation(req, tenantId)` to your auth-guard module — returns either null (passthrough) or a 423 NextResponse.
4. Inject it into every guard that handles mutations (`withOrgAdmin`, `withPartnerRole`, `withSupportAccessOr`, etc.).
5. Mock `getTenantLifecycle` in pre-existing route tests that use queue-shifting DB mocks (one-line `vi.mock` per test file).
6. Remove any per-route `ensureCanWrite` calls that the guard now handles — they become redundant.
7. If you have a cascade worker that flips status (e.g. `suspended -> locked` after N days), confirm the guard automatically picks up the new state at the next request.

## NOTES
- For very-fine-grained lifecycle gating (e.g. "this org can read services but not invoices when in `view_only_billing_dispute` mode"), keep the routine read-block at the guard layer but accept that some routes will need extra logic. Don't try to encode every business rule in the guard — only the cross-cutting ones.
- This pattern composes naturally with role checks. The guard does role first, then lifecycle, then handler. If role fails the lifecycle check is skipped (no DB hit), keeping unauthorized requests cheap.
- The "missing tenant returns null" semantics matters: the guard should NOT 404 — the route handler often has richer 404 logic (with org/partner context). Let the guard be silent when the tenant isn't there.
- Pair with structured logging: log `{ tenantId, status, blocked: true }` when the guard returns 423 so the support team can correlate "why did my customer get locked out at 3am" with the cascade worker run.
- Test interaction: queue-shifting DB mocks in pre-existing route tests will need a `vi.mock("@/lib/auth/partner-lifecycle", () => ({ getTenantLifecycle: vi.fn().mockResolvedValue(null) }))` stub so the new guard hop short-circuits without consuming a queue slot. (See LL-G `vitest-queue-shift-mocks-need-stubs-for-new-guard-hops`.)
