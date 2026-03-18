---
name: apply-practice
description: Apply a specific best practice from BP to a target repository
---

You are applying a specific best practice from the BP knowledge base to a target repository.

## Step 1: Identify inputs

You need two things:
1. **Practice slug** -- e.g., `testing/vitest-monorepo-config` or `safety/read-only-first-rule`
2. **Target repo** -- e.g., `60k-mono` (assumes `C:\Github\<repo-name>`)

Ask the user for any missing inputs.

## Step 2: Load the practice

Read `C:\Github\BP\practices\<concern>\<slug>.md` to get the full practice entry.

## Step 3: Run CHECK steps

Execute each CHECK item against the target repo to verify the practice isn't already applied.

- If ALL checks pass: inform the user "This practice is already applied to <repo-name>" and stop.
- If SOME checks pass: inform the user which parts are already in place and which are missing. Ask if they want to proceed with the missing parts only.
- If NO checks pass: proceed to implementation.

## Step 4: Follow IMPLEMENT steps

Execute each step in the IMPLEMENT section against the target repo. For each step:
1. Show what you're about to do
2. Execute the change
3. Verify the change was applied correctly

## Step 5: Validate

Re-run the CHECK steps to confirm all checks now pass.

## Step 6: Report

Output:
- Which steps were applied
- Which checks now pass
- Any manual follow-up needed (e.g., "run tests to verify", "restart dev server")
