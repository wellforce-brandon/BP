---
concern: deployment
tech: [stripe, postgres, nextjs, typescript]
priority: recommended
source-repo: vigilis
applies-to: [stripe, stripe-connect, multi-tenant]
---
# Two-endpoint Stripe webhook routing with composite-key idempotency

## PATTERN
For platforms that run both their own Stripe billing AND Stripe Connect (charging customers via connected accounts), register **two separate webhook endpoints** in Stripe — one for platform events, one for Connect events — and dedupe both through a single `webhook_events` table keyed on `(stripe_event_id, endpoint_source)`.

The two endpoints have:
- **Different signing secrets** (`STRIPE_WEBHOOK_SECRET_PLATFORM` and `STRIPE_WEBHOOK_SECRET_CONNECT`).
- **Different event subscriptions** (subscriptions/invoices on the platform endpoint; `account.updated`, `account.application.{authorized,deauthorized}`, and any per-org subscription/invoice events on the connect endpoint).
- **The same idempotency table**, distinguished by an `endpoint_source` enum.

The composite unique key `(stripe_event_id, endpoint_source)` is necessary because Stripe's `event.id` collides across endpoints when the same underlying event reaches both contexts. De-duping on `event.id` alone would drop legitimate Connect events when both endpoints fire.

Each route does:
```
verify signature with the right secret
  -> INSERT ... ON CONFLICT DO NOTHING on (stripe_event_id, endpoint_source)
  -> if conflict, return 200 with { duplicate: true }
  -> else dispatch by event.type, then mark webhook_events.status = 'completed'
  -> on error, mark webhook_events.status = 'failed' and return 500
```

## WHY
- **Per-endpoint signing is required by Stripe.** A single endpoint cannot verify both platform-mode and connect-mode events — Stripe signs them with different secrets.
- **Composite idempotency key handles event ID collisions.** Without `endpoint_source` in the key, a Connect-mode duplicate of a platform event would be silently dropped.
- **Dispatch logic stays per-endpoint.** Mixing platform and Connect handlers in one route forces every dispatch arm to first check `event.account` to disambiguate; with two endpoints that disambiguation is implicit.
- **Failure isolation.** If Stripe disables one endpoint due to repeated 5xx, the other keeps working. A combined endpoint takes both flows down together.
- **Easier replay.** `stripe events resend evt_xxx --endpoint we_yyy` targets the right handler without ambiguity.

## EXAMPLE
Schema (`src/lib/db/schema/webhook-events.ts`):
```ts
export const WEBHOOK_ENDPOINT_SOURCES = ["platform", "connect"] as const;
export const webhookEvents = pgTable("webhook_events", {
  id: id(),
  stripeEventId: text("stripe_event_id").notNull(),
  endpointSource: text("endpoint_source", { enum: WEBHOOK_ENDPOINT_SOURCES }).notNull(),
  receivedAt: timestamp("received_at", { withTimezone: true }).notNull().defaultNow(),
  processedAt: timestamp("processed_at", { withTimezone: true }),
  status: text("status", { enum: ["pending","processing","completed","failed"] }).notNull().default("pending"),
  payload: jsonb("payload").notNull(),
}, (t) => [
  unique("webhook_events_event_endpoint_unq").on(t.stripeEventId, t.endpointSource),
]);
```

Claim helper (`src/lib/integrations/stripe/webhooks.ts`):
```ts
export async function claimWebhookEvent(
  event: Stripe.Event,
  endpointSource: StripeWebhookEndpointSource,
): Promise<"claimed" | "duplicate"> {
  const result = await db.insert(webhookEvents)
    .values({ stripeEventId: event.id, endpointSource, status: "processing", payload: event })
    .onConflictDoNothing({
      target: [webhookEvents.stripeEventId, webhookEvents.endpointSource],
    })
    .returning({ id: webhookEvents.id });
  return result.length === 0 ? "duplicate" : "claimed";
}
```

Route (`src/app/api/webhooks/stripe/platform/route.ts`):
```ts
export async function POST(req: NextRequest) {
  const body = await req.text();
  const event = constructEvent(body, sig, "platform");
  const claim = await claimWebhookEvent(event, "platform");
  if (claim === "duplicate") return NextResponse.json({ received: true, duplicate: true });
  try {
    // dispatch by event.type ...
    await markWebhookEventCompleted(event.id, "platform");
    return NextResponse.json({ received: true });
  } catch {
    await markWebhookEventFailed(event.id, "platform");
    return NextResponse.json({ error: "Processing failed" }, { status: 500 });
  }
}
```

## CHECK
- [ ] `webhook_events` (or equivalent) table has unique constraint on `(stripe_event_id, endpoint_source)`.
- [ ] Two separate route files exist for platform and connect endpoints.
- [ ] Each route reads its signing secret from a distinct env var.
- [ ] Routes use `request.text()` before signature verification, not `request.json()`.
- [ ] Dedup uses `INSERT ... ON CONFLICT DO NOTHING`, not a "SELECT then INSERT" pattern (race-prone).
- [ ] On dispatch error, the webhook_events row is marked `failed` (not deleted).

## IMPLEMENT
1. Create the `webhook_events` schema with the composite unique constraint.
2. Add `STRIPE_WEBHOOK_SECRET_PLATFORM` and `STRIPE_WEBHOOK_SECRET_CONNECT` to your secrets manager.
3. In Stripe dashboard (or via API), create two endpoints — one for platform events, one with `connect: true`.
4. Build a `claimWebhookEvent(event, endpointSource)` helper using Drizzle's `.onConflictDoNothing({ target: [...] })`.
5. Implement the two route handlers; share dispatch logic only at the per-event-type-handler level, not at the route level.
6. Add `markWebhookEventCompleted` / `markWebhookEventFailed` for status tracking.
7. Test idempotency with `stripe events resend <event_id> --endpoint <we_id>` against both endpoints.

## NOTES
- For platforms that don't use Connect, a single endpoint is fine — keep the schema simple. The composite key is overkill for single-endpoint setups.
- When migrating from a single legacy endpoint, keep it registered as a third source value (`legacy`) so old events still process while you flip new traffic over.
- Pair with `oauth-state-tenant-encoding` if your Connect onboarding flow needs to attribute events to specific tenants — Connect events arrive with an `event.account` field that matches `partners.stripeConnectAccountId`.
- Stripe's own "official" idempotency guidance (`event.id` only) breaks if the same event reaches two endpoints — composite keys are the right granularity for multi-endpoint setups.
