---
concern: versioning
tech: [node, typescript]
priority: recommended
source-repo: MCP
applies-to: [node, typescript]
---
# Major.Minor.Patch.Build Versioning

## PATTERN
Use a four-segment version format (`Major.Minor.Patch.Build`) where the Build number increments on every single commit. This provides finer granularity than standard semver by tracking every change, not just releases.

| Segment | When to increment | Resets |
|---------|-------------------|--------|
| **Major** | Breaking changes -- API contracts, schema migrations, auth flow changes | Minor, Patch, Build -> 0 |
| **Minor** | New features -- new pages, endpoints, integrations | Patch, Build -> 0 |
| **Patch** | Bug fixes, security patches, perf improvements | Build -> 0 |
| **Build** | Every commit -- docs, refactors, config, tests, chores | Nothing |

## WHY
Standard semver (`1.2.3`) only changes on meaningful releases, making it hard to correlate a deployed version to a specific commit. The Build segment provides a monotonically increasing counter that changes with every commit, making it trivial to identify exact deployed versions and detect stale deployments.

## EXAMPLE
From MCP (`.claude/rules/commit-changelog.md`):
```markdown
Rules:
- The Build number increments on every single commit, no exceptions.
- When a higher segment increments, all lower segments reset to 0.
- If a commit includes both a feature and a bug fix, use the highest
  applicable bump (Minor in that case).
- NEVER bump Major autonomously. Always ask the user for guidance.
- If unsure between Minor and Patch, ask the user.
```

Version progression:
```
1.2.3.4 -> 1.2.3.5  (docs change, Build bump)
1.2.3.5 -> 1.2.4.0  (bug fix, Patch bump, Build resets)
1.2.4.0 -> 1.3.0.0  (new feature, Minor bump)
```

## CHECK
How to verify if a repo already follows this:
- [ ] `package.json` version field uses 4-segment format (e.g., `1.0.0.1`)
- [ ] A commit/changelog rule exists in `.claude/rules/` enforcing version bumps
- [ ] CHANGELOG.md exists with Keep a Changelog format

## IMPLEMENT
1. Update `package.json` version to 4-segment format (e.g., `0.1.0.0`)
2. Create `.claude/rules/commit-changelog.md` with the version bump rules
3. Create `CHANGELOG.md` with `[Unreleased]` section
4. Add "stage both files" reminder to the commit rule

## NOTES
- The "NEVER bump Major autonomously" rule is important -- Major bumps are a human decision
- This pairs with changelog enforcement (see `versioning/changelog-enforcement`)
- Works with any package.json project; not tied to npm publishing
