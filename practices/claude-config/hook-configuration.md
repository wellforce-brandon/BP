---
concern: claude-config
tech: [claude-code]
priority: recommended
source-repo: 60k-mono
applies-to: [all]
---
# Hook Configuration for Automated Enforcement

## PATTERN
Use Claude Code hooks in `.claude/settings.json` to automate enforcement of repo standards. Key hook events: PreToolUse (gate dangerous operations), PostToolUse (auto-lint after writes), Stop (audible bell when done), Notification (alert on permission prompts).

## WHY
Written instructions in CLAUDE.md can be overlooked. Hooks enforce rules at execution time -- if a hook rejects an action, it cannot proceed. This makes critical safety rules (like permissions gates) and quality rules (like auto-formatting) mechanically enforced rather than advisory.

## EXAMPLE
From 60k-mono (`.claude/settings.json`):
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash(git commit*)",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Committing changes...'"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo '\\a'"
          }
        ]
      }
    ],
    "Notification": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo '\\a'"
          }
        ]
      }
    ]
  }
}
```

From tech-assistant -- PreToolUse hook to enforce permissions gate:
```json
{
  "matcher": "Bash(*MgServicePrincipalAppRoleAssignment*|*MgDirectoryRoleMember*|*MgApplication*RequiredResourceAccess*)",
  "hooks": [
    {
      "type": "command",
      "command": "bash .claude/hooks/check-permissions-gate.sh"
    }
  ]
}
```

## CHECK
How to verify if a repo already follows this:
- [ ] `.claude/settings.json` exists with a `hooks` section
- [ ] Stop hook with audible bell is configured
- [ ] Notification hook with audible bell is configured
- [ ] PreToolUse hooks exist for any critical safety gates

## IMPLEMENT
1. Create or update `.claude/settings.json` with basic hook structure
2. Add Stop and Notification hooks with `echo '\\a'` for audible alerts
3. Identify any repo-specific safety gates that need PreToolUse enforcement
4. Add PostToolUse hooks for auto-formatting if the repo uses Biome/ESLint

## NOTES
- Hook types: `command` (shell), `http` (POST to URL), `prompt` (single-turn LLM yes/no), `agent` (multi-turn subagent)
- Hook events: PreToolUse, PostToolUse, PostToolUseFailure, Stop, Notification, SubagentStart, SubagentStop, PreCompact, SessionStart, SessionEnd, UserPromptSubmit, PermissionRequest
- `settings.local.json` supports `disableAllHooks` kill switch for debugging
