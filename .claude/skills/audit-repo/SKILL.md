---
name: audit-repo
description: Audit a repo against applicable best practices from the BP knowledge base
---

You are auditing a repository against the BP best practices knowledge base at `C:\Github\BP`.

## Step 1: Identify the target repo

Ask the user which repo to audit if not already specified. The repo should be in `C:\Github\<repo-name>`.

## Step 2: Load repo inventory

Read `C:\Github\BP\.claude\references\repo-inventory.md` to find the target repo's tags (tech stack, frameworks, characteristics).

## Step 3: Determine applicable practices

1. Read the master `C:\Github\BP\llms.txt` to get all concern categories
2. For each concern, read its `llms.txt` index
3. For each practice entry, check if its `applies-to` tags match the repo's tags
4. All FOUNDATIONAL entries apply regardless of tags

## Step 4: Run CHECK steps

For each applicable practice:
1. Read the practice entry file
2. Execute each item in the CHECK section against the target repo
3. Mark each check as PASS or FAIL
4. Record findings

## Step 5: Generate report

Write a report to `C:\Github\BP\audits\<repo-name>.md` with this format:

```markdown
# BP Audit: <repo-name>
Date: <YYYY-MM-DD>

## Summary
- Applicable practices: X
- Passing: Y
- Failing: Z
- Score: Y/X (percentage%)

## FOUNDATIONAL (must-fix)
### [Practice Title](../practices/<concern>/<slug>.md)
- Status: PASS/FAIL
- Details: What was found
- Missing: What needs to be done (if FAIL)

## RECOMMENDED
### [Practice Title](../practices/<concern>/<slug>.md)
- Status: PASS/FAIL
- Details: What was found

## OPTIONAL
### [Practice Title](../practices/<concern>/<slug>.md)
- Status: PASS/FAIL
- Details: What was found
```

## Step 6: Present results

Output the summary to the user:
- Total score
- List of failing FOUNDATIONAL practices (these should be fixed first)
- List of failing RECOMMENDED practices
- Path to the full report file
