---
concern: claude-config
tech: [claude-code, git]
priority: recommended
source-repo: vigilis
applies-to: [phase-based-projects, large-features]
---
# Multi-Agent Concurrent Development via Git Worktree Isolation

## PATTERN
For large, phase-based features that decompose into independent sub-phases, run multiple Claude agents in parallel — each in its own `git worktree` rooted at `.claude/worktrees/<phase-id>` — and merge results back to `main` with sequential `git merge --no-ff` calls. The worktree isolation prevents cross-agent file contention; the no-fast-forward merges produce a readable history that maps 1:1 to phase boundaries.

## WHY
A serial agent run on a 5-phase feature spends most of its time waiting on filesystem I/O and tool round-trips that are independent across phases. Parallelizing across worktrees collapses wall-clock time roughly N:1 with N concurrent agents, and the worktree boundary prevents the most common failure mode of parallel work — two agents touching the same file. The merge story is also cleaner than rebase-onto-main churn: each phase produces one merge commit, easy to revert in isolation.

## EXAMPLE
From Vigilis Phase 2 (multi-broker auth + admin scope, 5 sub-phases shipped concurrently):

```bash
# 1. Spin up one worktree per independent sub-phase from a clean main.
git worktree add .claude/worktrees/phase-2bc -b phase-2bc main
git worktree add .claude/worktrees/phase-2d  -b phase-2d  main
git worktree add .claude/worktrees/phase-2gh -b phase-2gh main
git worktree add .claude/worktrees/phase-2i  -b phase-2i  main
git worktree add .claude/worktrees/phase-2j  -b phase-2j  main

# 2. Launch one agent per worktree, each scoped to its own task plan.
#    Each agent commits inside its worktree branch.

# 3. Merge sequentially into main with explicit merge commits.
git checkout main
for branch in phase-2bc phase-2d phase-2gh phase-2i phase-2j; do
  git merge --no-ff "$branch" -m "Merge $branch into main"
done

# 4. Tear down worktrees and branches once merged.
git worktree remove .claude/worktrees/phase-2bc
git branch -d phase-2bc
# ...repeat per phase
```

Decomposition rules that made this work:
- Each sub-phase owned a disjoint set of paths (different `src/lib/auth/*` files, different schema files, different API route prefixes).
- Shared files (`templates.ts`, `_journal.json`, `package.json`) had a designated owning phase; other phases waited or appended via post-merge edits.
- Migration `when` timestamps were pre-assigned per phase so journal merges were deterministic (LL-G `drizzle-kit-migrate-silent-skip`).

## CHECK
How to verify a repo is set up for this:
- [ ] `.claude/worktrees/` is in `.gitignore` (worktrees are ephemeral; never commit them).
- [ ] Task plans in `tasks/` enumerate phase boundaries with explicit owned paths.
- [ ] CLAUDE.md or a rules file documents the per-phase migration timestamp convention if Drizzle/Prisma is in use.
- [ ] CI runs against `main` post-merge, not against agent branches (catches integration regressions).

## IMPLEMENT
1. Add `.claude/worktrees/` to `.gitignore`.
2. Author the master plan (`tasks/<feature>.md`) and break it into independent sub-phases with explicit "owns these paths" sections.
3. For each sub-phase, create a worktree and dispatch a Claude agent with its scoped plan as the prompt.
4. After each agent completes, switch to `main` and `git merge --no-ff <phase-branch>`. Resolve conflicts immediately while context is fresh.
5. Verify integration: run the full test suite + type check on `main` after each merge, not just at the end.
6. Remove worktrees and short-lived branches once merged.

## NOTES
- **File contention is the #1 failure mode.** When two phases must touch the same file (e.g., a shared `templates.ts` or a single migration journal), assign it to one phase and have others rebase or append after that phase merges. Don't try to merge concurrent edits to the same file across worktrees.
- **Watch for parent-tree leakage.** Agents occasionally write to the parent repo working tree instead of their worktree (config drift, IDE state). Run `git status` in the parent before merging; discard stray changes with `git checkout -- <files>` and `git clean -fd` to keep merges clean.
- **Migration journals need pre-assigned timestamps.** With Drizzle, two phases generating migrations independently will collide on `_journal.json` ordering and one will be silently skipped. Assign each phase a reserved `when` window upfront.
- **Cap concurrency at 4-6 agents.** Higher counts amplify integration debugging cost and exceed the human reviewer's ability to rationalize the merged diff.
- **Worktree cleanup gotcha (Windows).** `git worktree remove` can fail on long paths or open handles; fall back to `rmdir /s` or PowerShell `Remove-Item -Recurse -Force` and `git worktree prune`.
- Real result: Vigilis Phase 2 collapsed an estimated 5-day serial agent run into roughly 1 day of wall-clock time across 5 concurrent agents, with a single mid-merge conflict (email templates) resolved manually in under 10 minutes.
