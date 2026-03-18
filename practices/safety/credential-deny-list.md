---
concern: safety
tech: [claude-code]
priority: foundational
source-repo: external
applies-to: [all]
---
# Credential Path Deny-List in Settings

## PATTERN
Configure explicit `deny` rules in `.claude/settings.json` blocking reads of sensitive paths: SSH keys, cloud credentials, Docker configs, npm tokens, git credentials, and shell config files. This is defense-in-depth -- even with sandboxing, explicit deny rules prevent accidental credential exfiltration.

## WHY
AI assistants can be tricked via prompt injection in code comments or file contents into reading and exposing credentials. A deny-list in settings.json mechanically blocks access to sensitive paths regardless of what the AI is asked to do. This is the AI-specific equivalent of filesystem permissions.

## EXAMPLE
```json
{
  "permissions": {
    "deny": [
      "Read(~/.ssh/**)",
      "Read(~/.aws/**)",
      "Read(~/.azure/**)",
      "Read(~/.kube/**)",
      "Read(~/.docker/config.json)",
      "Read(~/.npmrc)",
      "Read(~/.git-credentials)",
      "Read(~/.config/gh/**)",
      "Edit(~/.bashrc)",
      "Edit(~/.zshrc)",
      "Edit(~/.profile)",
      "Edit(~/.bash_profile)"
    ]
  }
}
```

## CHECK
- [ ] `.claude/settings.json` has a `deny` array in permissions
- [ ] Deny list covers SSH, AWS, Azure, Kube, Docker, npm, git credential paths
- [ ] Shell config files (bashrc, zshrc) are denied for edits (backdoor prevention)

## IMPLEMENT
1. Add `deny` array to `.claude/settings.json` permissions
2. Include all standard credential paths for your platform
3. Add shell config deny rules to prevent backdoor insertion
4. Test by attempting to read a denied path -- should be blocked

## NOTES
- Source: trailofbits/claude-code-config, awesome-copilot hooks
- Windows equivalent paths: `C:\Users\*\.ssh\**`, etc.
- The deny-list complements RULE 0 (read-only-first) -- RULE 0 is behavioral, deny-list is mechanical
- Consider also denying `.env` files: `"Read(**/.env)"`
