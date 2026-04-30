---
concern: deployment
tech: [oauth, nextjs, typescript]
priority: recommended
source-repo: vigilis
applies-to: [oauth, multi-tenant, stripe-connect, github-apps, slack-oauth, atlassian-connect]
---
# Encode tenant ID in the OAuth `state` parameter for single-redirect-URI flows

## PATTERN
For multi-tenant systems where tenants connect external accounts (Stripe Connect, GitHub Apps, Slack OAuth, Atlassian Connect, custom SSO), most "platform OAuth" providers register a **single redirect URI** per platform. Wildcards aren't allowed, so per-tenant path segments (`/api/tenant/[tenantId]/oauth/callback`) cannot be registered. Instead:

1. Generate a high-entropy random nonce on the start route.
2. Encode tenant ID + nonce into the OAuth `state` parameter as `${tenantId}.${nonce}`.
3. Register one fixed callback URI (e.g. `/api/oauth/callback`) with the provider.
4. The callback parses `state`, splits on `.`, extracts the tenant ID, **re-verifies the session has admin role on that tenant**, and only then exchanges the code and persists the connection.

The session role re-check is the security pivot: `state` is attacker-controllable (it's in the URL the user sees during the redirect), so you can't trust the encoded ID without verifying ownership.

## WHY
- **Provider constraint, not preference.** Stripe Connect, GitHub Apps, Slack, and most "platform" OAuth flows enforce single-URI registration. You cannot work around it; this pattern is the only safe path.
- **The single-URI requirement isn't documented prominently.** Most engineers discover it by registering a per-tenant URI, getting a `redirect_uri_mismatch` error, then suspecting their dashboard config. The encode-in-state pattern is the answer.
- **State serves dual purpose.** It carries tenant ID AND CSRF nonce in one field — providers don't add nonce slots, so combining is the standard.
- **Re-verifying on callback closes the security gap.** Without it, a malicious user could craft a callback URL with another tenant's ID in `state` and hijack the connection. The role check on the session blocks this.
- **Avoids cookie-based tenant tracking.** Sticking tenant ID in a cookie before redirect "works" but breaks if the user starts the flow in one browser tab and finishes in another, or if their session times out mid-flow.

## EXAMPLE
Start route encodes the state (`src/app/api/partner/[partnerId]/billing/connect/start/route.ts`):
```ts
import { randomBytes } from "node:crypto";

export const POST = withPartnerAdmin(async (req, { partnerId }) => {
  if (!isStripeConnectConfigured()) return apiError("not configured", 503);

  const random = randomBytes(24).toString("hex");
  const state = `${partnerId}.${random}`; // tenant ID + CSRF nonce

  const url = buildConnectAuthorizeUrl({
    state,
    email: partner.recoveryContactEmail,
  }); // helper sets redirect_uri to STRIPE_CONNECT_REDIRECT_URI

  return apiSuccess({ url, state });
});
```

Single fixed callback (`src/app/api/billing/connect/callback/route.ts`):
```ts
export async function GET(req: NextRequest) {
  const session = await auth.api.getSession({ headers: req.headers });
  if (!session) return NextResponse.redirect("/login");

  const state = req.nextUrl.searchParams.get("state");
  if (!state || !state.includes(".")) {
    return NextResponse.redirect("/?connect_error=invalid_state");
  }
  const partnerId = state.split(".")[0];

  // CRITICAL: re-verify session role on the encoded tenant.
  const role = await getActivePartnerRole(session.user.id, partnerId);
  if (!role || !hasPartnerRole([{ partnerId, role }], partnerId, "partner_admin")) {
    return NextResponse.redirect(
      `/admin/partner/${partnerId}/settings?connect_error=forbidden`,
    );
  }

  const code = req.nextUrl.searchParams.get("code");
  const exchanged = await exchangeAuthorizationCode(code);
  await db.update(partners).set({
    stripeConnectAccountId: exchanged.stripeUserId,
  }).where(eq(partners.id, partnerId));

  return NextResponse.redirect(
    `/admin/partner/${partnerId}/settings?connect=success`,
  );
}
```

Buildable env config:
```bash
# Doppler prd
STRIPE_CONNECT_CLIENT_ID=ca_xxxxxx
STRIPE_CONNECT_REDIRECT_URI=https://your-app.com/api/billing/connect/callback
```

## CHECK
- [ ] OAuth `state` is generated server-side using `crypto.randomBytes` or equivalent (NOT `Math.random`).
- [ ] Tenant ID and a high-entropy nonce are concatenated with a delimiter into `state`.
- [ ] The callback route is at a fixed path with no `[tenantId]` segment.
- [ ] The callback re-verifies session role ownership of the encoded tenant ID before persisting.
- [ ] The callback redirects to a tenant-scoped page on success/error so the user lands somewhere useful.
- [ ] The fixed redirect URI is registered with the OAuth provider AND set in your env vars.
- [ ] OAuth provider's "redirect_uri" parameter on the start request matches the registered URI exactly.

## IMPLEMENT
1. Pick the canonical fixed callback path (e.g. `/api/oauth/callback` or `/api/billing/connect/callback`).
2. Register that exact URI in the OAuth provider's dashboard (Stripe: `dashboard.stripe.com/settings/connect/onboarding-options`; GitHub: app settings -> Callback URL).
3. Set the same URI in your env vars (`STRIPE_CONNECT_REDIRECT_URI`, `GITHUB_OAUTH_CALLBACK`, etc).
4. Build a `buildAuthorizeUrl({ state, email? })` helper that reads the URI from env and the client_id from env, and returns the full provider URL.
5. Move/refactor any existing per-tenant callback to the fixed path; update tenant ID extraction to read from `state.split(".")[0]`.
6. Add the role re-verification step. Test it with a session that has admin role on tenant A but tries to use a state encoding tenant B.
7. Document the env-driven activation: routes return 503 when `STRIPE_CONNECT_CLIENT_ID` etc. are unset.

## NOTES
- The same pattern applies to **deauthorization callbacks** if the provider uses one — encode the tenant ID in any extra parameter the provider supports.
- For providers that DO allow multiple registered redirect URIs (some custom IDPs), prefer one URI per tenant only if you have fewer than ~10 tenants. Above that, the tenant-in-state pattern scales better and matches the "platform OAuth" mental model.
- **Don't try to use the OAuth `state` for cross-domain auth.** It's a request-level token, not a session token. After successful exchange, drive the user back to your own session.
- Pair with the BP "two-endpoint Stripe webhook idempotency" pattern when the OAuth flow culminates in webhooks (e.g. Stripe Connect's `account.updated` event after onboarding completes).
- Pair with the BP "host-mode-rollout" entry — the same env-var-presence-as-activation idea applies to both subdomain rollout and OAuth provider provisioning.
