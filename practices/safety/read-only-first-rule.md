---
concern: safety
tech: [claude-code, powershell, graph-api]
priority: foundational
source-repo: tech-assistant
applies-to: [all]
---
# Read-Only First Rule (RULE 0)

## PATTERN
Every interaction follows a strict read-before-write discipline. Before any modification, run read-only commands first to understand current state. Modifications only happen after the user sees and approves the current state.

Three safety tiers:
1. **Always Safe** -- `Get-*`, `Test-*`, `Resolve-*`, read-only queries, diagnostic commands
2. **Requires User Approval** -- `Set-*`, `Stop-*`, `Start-*`, `Restart-*`, `Remove-*`, any state change
3. **Never Without Explicit Request** -- destructive operations like `Remove-Item -Recurse -Force`, credential resets, disk formatting

## WHY
Accidental modifications in production environments (especially M365 tenants, infrastructure, or shared services) can be catastrophic and difficult to reverse. The read-only-first pattern creates a natural checkpoint where the user can verify intent before any changes take effect.

## EXAMPLE
From tech-assistant (`CLAUDE.md`):
```markdown
## The Prime Directive

**Gather information before taking action. Read-only commands first.
Modifications only with user approval.**

When in doubt: "What's the safest way to diagnose this without
making changes?"
```

Applied to M365 scripting:
```powershell
# STEP 1: Read-only -- show what would be affected
$externalUsers = Get-MgGroupMember -GroupId $groupId | Where-Object {
    $_.AdditionalProperties.userType -eq 'Guest'
}
Write-Host "Found $($externalUsers.Count) external users in group"
$externalUsers | Format-Table DisplayName, Mail

# STEP 2: Only after user confirms, proceed with removal
# (never in the same script execution as step 1)
```

## CHECK
How to verify if a repo already follows this:
- [ ] CLAUDE.md contains a RULE 0 or "read-only first" directive
- [ ] Safety tiers are defined (always safe, requires approval, never without request)
- [ ] Scripts that modify state are separated from scripts that read state

## IMPLEMENT
1. Add a "Safety Protocols" or "RULE 0" section to CLAUDE.md
2. Define three tiers of commands for the repo's technology stack
3. For M365/Graph repos: always generate a read-only script first, then a modification script after user review
4. Consider PreToolUse hooks to gate dangerous operations

## NOTES
- This pattern is especially critical for MSP/multi-tenant environments
- The `claude-instructions.md` in tech-assistant enforces this for all M365 operations
- Complement with PreToolUse hooks for mechanical enforcement
