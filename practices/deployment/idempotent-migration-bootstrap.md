---
concern: deployment
tech: [typescript, drizzle, postgres, sql]
priority: recommended
source-repo: vigilis
applies-to: [migrations, bootstrap-scripts]
---
# Idempotent Migration Bootstrap Scripts

## PATTERN

Schema migrations describe column-level changes. Bootstrap data (seed rows, founding tenants, default tiers, system users) does not belong in column migrations because it is data, not structure. But it still needs to run on every fresh environment AND be safe to re-run on existing environments.

The pattern: write bootstrap as a separate idempotent script (`scripts/migrations/NN-description.ts`) that:

1. Uses `INSERT ... ON CONFLICT (key) DO NOTHING` (or `DO UPDATE` if the row should converge to a known state) so re-running is a no-op.
2. Is invoked from CI/deploy after `drizzle-kit migrate` succeeds, in a numbered sequence so order is deterministic.
3. Logs every action with `inserted: N, skipped: N` so deploy logs answer "did this actually do something this time?"
4. Is checked into the repo and code-reviewed like any other migration.
5. Uses primary keys derived from a stable, meaningful identifier (e.g. `partner_socium`) rather than auto-generated cuids, so re-runs match the existing row.

## WHY

A pure schema migration cannot encode "the founding partner row exists". Three common alternatives are all worse:

- **Manual SQL run once in production.** Easy to forget, easy to skip on a fresh staging DB, easy to lose during disaster recovery.
- **Embed seeds in a Drizzle migration.** Mixes structural and data concerns; if you need to roll the migration forward in a different shape (or replay it on a clone), the data half breaks.
- **Run a one-shot seed script and trust it never re-runs.** Inevitable that it does -- a teammate runs it on a fresh branch DB, a CI clone, a recovered backup -- and now you have a duplicate-key crash or a corrupted row.

Idempotent bootstrap scripts solve all three. They are safe to re-run, cheap to re-run on every deploy, and become the documented source of truth for "what data must exist for the app to function".

## EXAMPLE

From vigilis (`scripts/migrations/01-create-socium-partner.ts`):

```typescript
import { db } from "@/lib/db";
import { partners, partnerBillingTiers } from "@/lib/db/schema";
import { logger } from "@/lib/logger";
import { sql } from "drizzle-orm";

const log = logger.child({ script: "01-create-socium-partner" });

async function run() {
  // Idempotent: ON CONFLICT DO NOTHING means re-running is safe.
  const tierResult = await db.execute(sql`
    INSERT INTO partner_billing_tiers (id, name, price_cents, currency)
    VALUES ('tier_flat', 'Flat $50/mo', 5000, 'USD')
    ON CONFLICT (id) DO NOTHING
    RETURNING id
  `);

  const partnerResult = await db.execute(sql`
    INSERT INTO partners (id, display_name, status, billing_tier_id, created_at, updated_at)
    VALUES ('partner_socium', 'Vigilis', 'active', 'tier_flat', now(), now())
    ON CONFLICT (id) DO NOTHING
    RETURNING id
  `);

  log.info(
    {
      action: "bootstrap.complete",
      tier_inserted: tierResult.rowCount,
      partner_inserted: partnerResult.rowCount,
    },
    `Bootstrap finished: tier=${tierResult.rowCount}, partner=${partnerResult.rowCount}`,
  );
}

run().catch((err) => {
  log.error({ err }, "Bootstrap failed");
  process.exit(1);
});
```

Invoked in deploy as `pnpm db:migrate && pnpm tsx scripts/migrations/01-create-socium-partner.ts`. First run on a fresh DB inserts both rows; every subsequent run logs `tier_inserted=0, partner_inserted=0` and exits cleanly.

## CHECK

How to verify if a repo follows this pattern:
- [ ] Bootstrap data lives in `scripts/migrations/NN-*.ts` (or equivalent), not in Drizzle migration SQL.
- [ ] Every bootstrap script uses `ON CONFLICT DO NOTHING` or `ON CONFLICT DO UPDATE`.
- [ ] Script names are numbered so execution order is deterministic.
- [ ] Logs report inserted-vs-skipped counts.
- [ ] Deploy pipeline runs the scripts after `drizzle-kit migrate` (or equivalent).
- [ ] Re-running on an already-bootstrapped DB exits 0 with no row changes.

## IMPLEMENT

1. Create a `scripts/migrations/` directory parallel to your Drizzle `drizzle/` migrations folder.
2. Add a numbered TypeScript file per bootstrap concern (`01-founding-tenant.ts`, `02-default-billing-tiers.ts`, etc.).
3. Use stable, human-meaningful primary keys for bootstrap rows (`partner_socium`, `tier_flat`) rather than random ids.
4. Write each insert as `INSERT ... ON CONFLICT (id) DO NOTHING RETURNING id`, then log `rowCount`.
5. For rows that should converge to a known state (e.g. defaults that may drift), use `ON CONFLICT (id) DO UPDATE SET ...`.
6. Wire the runner into your deploy command: `pnpm db:migrate && pnpm bootstrap` where `bootstrap` runs the scripts in order.
7. Add a CI smoke test that runs the bootstrap script twice on a clean DB and asserts the second run inserts zero rows.

## NOTES

- For multi-environment seeds (different default tier in dev vs prod), gate inside the script via `process.env.NODE_ENV` rather than maintaining parallel scripts.
- This pattern composes with feature flags: a script can insert a flag row in `feature_flags` with a default `disabled` state, then a later migration flips it on for the rollout.
- Keep bootstrap scripts small and single-purpose. If one needs branching logic for "did the partner already exist?", that is a code smell -- prefer two scripts.
- Disaster recovery: when restoring from a logical backup that may or may not include the bootstrap rows, idempotent scripts mean "just run them all again" is a safe recovery step.
