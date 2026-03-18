---
concern: safety
tech: [claude-code, graph-api, powershell]
priority: recommended
source-repo: tech-assistant
applies-to: [graph-api, m365]
---
# Permissions Pass Gate (RULE 2)

## PATTERN
Privileged operations (modifying API permissions, assigning directory roles, granting admin consent) require a cryptographic verification step before execution. The user must provide a "permissions pass" that is validated against a tenant's PFX certificate before the operation proceeds.

Four mandatory steps:
1. **STOP** -- display target tenant, exact changes, whether admin consent is involved
2. **ASK** -- "Please provide the permissions pass to authorize this change"
3. **VERIFY** -- test the pass against the tenant PFX file
4. **PROCEED** -- only if pass is valid. Invalid = aborted, no retry.

## WHY
Graph API permission changes are high-impact and hard to audit after the fact. A single wrong `New-MgServicePrincipalAppRoleAssignment` can grant an app registration access to all mailboxes in a tenant. The permissions gate adds a human-verified checkpoint that cannot be bypassed, even if the user says "just do it."

## EXAMPLE
From tech-assistant (`CLAUDE.md`):
```markdown
## RULE 2 -- Permissions Pass Gate (MANDATORY -- NEVER SKIP)

**Before ANY operation that modifies Graph API permissions, assigns
directory roles, or grants admin consent: STOP.**

This applies to any script or command that uses:
- New-MgServicePrincipalAppRoleAssignment
- Update-MgApplication -RequiredResourceAccess
- Graph API calls to /appRoleAssignments

Required steps (ALL of them, EVERY time):
1. STOP -- display target tenant, exact changes
2. ASK -- "Please provide the permissions pass"
3. VERIFY -- test against tenant PFX
4. ONLY PROCEED if valid
```

Enforced mechanically with a PreToolUse hook:
```json
{
  "matcher": "Bash(*MgServicePrincipalAppRoleAssignment*)",
  "hooks": [{
    "type": "command",
    "command": "bash .claude/hooks/check-permissions-gate.sh"
  }]
}
```

## CHECK
How to verify if a repo already follows this:
- [ ] CLAUDE.md contains RULE 2 or equivalent permissions gate
- [ ] PreToolUse hook exists to enforce the gate at execution time
- [ ] Verification script exists (PFX-based or equivalent)

## IMPLEMENT
1. Add RULE 2 section to CLAUDE.md with the 4-step protocol
2. Create a verification script in `.claude/hooks/`
3. Add PreToolUse hook in `.claude/settings.json` matching dangerous Graph API patterns
4. Test the gate by attempting a permissions change and verifying it blocks

## NOTES
- This pattern is specific to M365/Graph API environments with multi-tenant management
- The concept (cryptographic gate for privileged operations) can be adapted to other domains
- The gate applies to runtime permission changes only, not initial app registration setup
