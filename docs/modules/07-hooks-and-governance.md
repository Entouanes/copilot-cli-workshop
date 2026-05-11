# Module 07: Hooks and Enterprise Governance

**Duration:** 120 minutes | **Prev:** [Module 06: Automation](06-automation.md) | **Next:** [Module 08: Advanced and Certification Prep](08-advanced-and-certification.md)

## Learning Objectives

- Understand all 8 hook types, their payloads, and when each fires
- Write shell scripts for `sessionStart`, `userPromptSubmitted`, and `preToolUse`
- Use `preToolUse` to block dangerous commands with audit logging
- Design a governance rollout strategy suitable for a development team

---

## 7.1 About Hooks (25 min)

Hooks run custom shell scripts at defined points in Copilot's execution. They work with both Copilot CLI and the Copilot cloud agent on GitHub.com.

Stored at: `.github/hooks/*.json` (committed to the repo, applies to everyone).

### All 8 Hook Types

| Hook | Fires when | Can block execution? | Typical use |
|---|---|---|---|
| `sessionStart` | New session begins or resumes | No | Welcome banner, audit log init, env validation |
| `sessionEnd` | Session terminates for any reason | No | Cleanup, final audit log entry |
| `userPromptSubmitted` | User submits a prompt | No | Log prompt metadata for auditing |
| `preToolUse` | Before any tool call (bash, edit, view) | **Yes** | Security policy enforcement |
| `postToolUse` | After tool completes (success or failure) | No | Log results, alert on failures, collect metrics |
| `agentStop` | Main agent finishes responding to a prompt | No | Post-response processing |
| `subagentStop` | A subagent completes and returns to parent | No | Subagent tracking |
| `errorOccurred` | Error during execution | No | Error notification (Slack, email) |

---

## 7.2 Hook Configuration Format (20 min)

```json
{
  "version": 1,
  "hooks": {
    "sessionStart": [
      {
        "type": "command",
        "bash": ".github/hooks/scripts/session-banner.sh",
        "powershell": ".github/hooks/scripts/session-banner.ps1",
        "cwd": ".",
        "env": { "LOG_LEVEL": "INFO" },
        "timeoutSec": 10
      }
    ]
  }
}
```

Property reference:

| Property | Required | Default | Description |
|---|---|---|---|
| `type` | Yes | None | Must be `"command"` |
| `bash` | Conditional | None | Path to bash script (Unix) |
| `powershell` | Conditional | None | Path to PowerShell script (Windows) |
| `cwd` | No | Session cwd | Working directory for the script |
| `env` | No | `{}` | Additional environment variables |
| `timeoutSec` | No | 30 | Max execution time in seconds |

Multiple hooks of the same type execute in array order:

```json
{
  "preToolUse": [
    { "type": "command", "bash": "./scripts/security-check.sh" },
    { "type": "command", "bash": "./scripts/audit-log.sh" }
  ]
}
```

### Input and Output Payloads

Every hook receives a JSON payload via **stdin**.

**`preToolUse` input:**

```json
{
  "timestamp": 1704614600000,
  "cwd": "/path/to/project",
  "toolName": "bash",
  "toolArgs": "{\"command\":\"rm -rf dist\",\"description\":\"Clean build directory\"}"
}
```

!!! warning
    `toolArgs` is a **JSON string** (not an object). Parse it separately with `jq -r '.toolArgs | fromjson'`.

**`preToolUse` output to block execution (the only hook that can deny):**

```json
{ "permissionDecision": "deny", "permissionDecisionReason": "Reason shown to Copilot and user" }
```

**`postToolUse` input:**

```json
{
  "toolName": "bash",
  "toolArgs": "{\"command\":\"npm test\"}",
  "toolResult": {
    "resultType": "success",
    "textResultForLlm": "All tests passed (15/15)"
  }
}
```

`resultType` values: `success` | `failure` | `denied`

---

## 7.3 Writing Production Hook Scripts (40 min)

### Session Banner (`sessionStart`)

```bash
#!/bin/bash
# .github/hooks/scripts/session-banner.sh
cat << 'EOF'
COPILOT CLI POLICY ACTIVE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
* Prompts and tool use are logged for auditing
* High-risk commands are blocked automatically
* If blocked, read the reason and adjust your prompt
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
EOF
exit 0
```

PowerShell equivalent (`.ps1`):

```powershell
$ErrorActionPreference = "Stop"
Write-Host @"
COPILOT CLI POLICY ACTIVE
* Prompts and tool use are logged for auditing
* High-risk commands are blocked automatically
"@
exit 0
```

### Prompt Audit Logger (`userPromptSubmitted`)

Logs metadata without logging full prompt content:

```bash
#!/bin/bash
# .github/hooks/scripts/log-prompt.sh
set -euo pipefail

INPUT="$(cat)"
TIMESTAMP_MS="$(echo "$INPUT" | jq -r '.timestamp // empty')"
CWD="$(echo "$INPUT" | jq -r '.cwd // empty')"

LOG_DIR=".github/hooks/logs"
mkdir -p "$LOG_DIR"
chmod 700 "$LOG_DIR"

jq -n \
  --arg ts "$TIMESTAMP_MS" \
  --arg cwd "$CWD" \
  '{event:"userPromptSubmitted", timestampMs:$ts, cwd:$cwd}' \
  >> "$LOG_DIR/audit.jsonl"

exit 0
```

To log the prompt text itself, apply token redaction first:

```bash
PROMPT="$(echo "$INPUT" | jq -r '.prompt // empty')"
REDACTED_PROMPT="$(echo "$PROMPT" | sed -E 's/ghp_[A-Za-z0-9]{20,}/[REDACTED]/g')"
```

### Security Policy Enforcer (`preToolUse`)

Full production script blocking privilege escalation, root filesystem destruction, and download-and-execute:

```bash
#!/bin/bash
# .github/hooks/scripts/pre-tool-policy.sh
set -euo pipefail

INPUT="$(cat)"
TOOL_NAME="$(echo "$INPUT" | jq -r '.toolName // empty')"
TOOL_ARGS_RAW="$(echo "$INPUT" | jq -r '.toolArgs // empty')"

LOG_DIR=".github/hooks/logs"
mkdir -p "$LOG_DIR"

# Redact sensitive values before logging
REDACTED="$(echo "$TOOL_ARGS_RAW" | \
  sed -E 's/ghp_[A-Za-z0-9]{20,}/[REDACTED_TOKEN]/g' | \
  sed -E 's/gho_[A-Za-z0-9]{20,}/[REDACTED_TOKEN]/g' | \
  sed -E 's/Bearer [A-Za-z0-9_\-\.]+/Bearer [REDACTED]/g' | \
  sed -E 's/--password[= ][^ ]+/--password=[REDACTED]/g')"

# Write audit entry
jq -n \
  --arg t "$TOOL_NAME" \
  --arg a "$REDACTED" \
  '{event:"preToolUse", toolName:$t, toolArgs:$a}' \
  >> "$LOG_DIR/audit.jsonl"

# Only enforce command-level rules for bash tool
[ "$TOOL_NAME" != "bash" ] && exit 0

# Parse the command string from toolArgs
COMMAND="$(echo "$TOOL_ARGS_RAW" | jq -r '.command // empty')"

deny() {
  local reason="$1"
  jq -n --arg cmd "$COMMAND" --arg r "$reason" \
    '{event:"policyDeny", toolName:"bash", command:$cmd, reason:$r}' \
    >> "$LOG_DIR/audit.jsonl"
  jq -n --arg r "$reason" '{permissionDecision:"deny", permissionDecisionReason:$r}'
  exit 0
}

# Rule 1: Block privilege escalation
echo "$COMMAND" | grep -qE '\b(sudo|su|runas)\b' \
  && deny "Privilege escalation requires manual approval."

# Rule 2: Block root filesystem destruction
echo "$COMMAND" | grep -qE 'rm\s+-rf\s*/($|\s)|rm\s+.*-rf\s*/($|\s)' \
  && deny "Destructive operations targeting the filesystem root are blocked."

# Rule 3: Block system format or wipe commands
echo "$COMMAND" | grep -qE '\b(mkfs|dd|format)\b' \
  && deny "System-level destructive operations are blocked."

# Rule 4: Block download-and-execute patterns
echo "$COMMAND" | grep -qE 'curl.*\|\s*(bash|sh)|wget.*\|\s*(bash|sh)' \
  && deny "Download-and-execute patterns require manual approval."

exit 0
```

### Error Notification (`errorOccurred`)

```bash
#!/bin/bash
# .github/hooks/scripts/notify-error.sh
INPUT="$(cat)"
ERROR_MSG="$(echo "$INPUT" | jq -r '.error.message // "unknown error"')"
WEBHOOK_URL="https://hooks.slack.com/services/YOUR/WEBHOOK/URL"

curl -s -X POST "$WEBHOOK_URL" \
  -H 'Content-Type: application/json' \
  -d "{\"text\":\"Copilot CLI error: $ERROR_MSG\"}"
exit 0
```

### Audit Log Inspection

```bash
# View recent entries
tail -n 50 .github/hooks/logs/audit.jsonl

# Show only denied tool calls
jq 'select(.event=="policyDeny")' .github/hooks/logs/audit.jsonl

# Count by tool name
jq -r '.toolName' .github/hooks/logs/audit.jsonl | sort | uniq -c | sort -rn
```

---

## 7.4 Enterprise Governance Design (35 min)

### Governance Rollout Strategy

GitHub's recommended phased approach:

**Phase 1: Logging only (weeks 1-2)**
Deploy hooks that log prompts and tool usage without denying anything. Review the audit trail to understand how developers actually use Copilot.

```json
{ "hooks": { "preToolUse": [{ "bash": "./scripts/audit-only.sh" }] } }
```

**Phase 2: Low-risk policies (weeks 3-4)**
Add deny rules for clearly unacceptable patterns (sudo, root rm, download-execute). Test on one team before org-wide rollout.

**Phase 3: Team-by-team expansion**
Roll out to additional teams one at a time, gathering feedback and adjusting rules.

**Phase 4: Risk-based policies**
Apply stricter policies to repos that touch production infrastructure or sensitive data.

### Org/Enterprise Admin Controls

| Control | How |
|---|---|
| Enable/disable Copilot CLI | Org settings: Copilot policy |
| BYOM (custom model provider) | `copilot help providers` |
| Org-level custom agents | `/agents/` in `.github-private` repo |
| Multi-account/GHE | `COPILOT_GH_HOST=github.myenterprise.com copilot` |
| Switch accounts mid-session | `/user list` then `/user switch` |

### What Hooks Cannot Enforce (Known Gaps as of v1.0.44)

- CLI cannot enforce org-level "MCP servers in Copilot" policy
- CLI cannot enforce org-level "MCP Registry URL" restrictions

Use hooks as the primary mechanism for MCP governance until these policies are available.

### Security Best Practices for Hook Scripts

- Keep execution time under 5 seconds (long hooks slow every tool call)
- Use asynchronous logging (append to files, not synchronous writes)
- Never log full prompt text without redacting tokens, passwords, and PII
- Make hook scripts executable: `chmod +x scripts/*.sh`
- Test hooks locally before deploying:
  ```bash
  echo '{"toolName":"bash","toolArgs":"{\"command\":\"ls\"}"}' | ./scripts/pre-tool-policy.sh
  ```
- Add `.github/hooks/logs/` to `.gitignore` (audit logs should not be committed)
- Set appropriate `timeoutSec` to prevent resource exhaustion

### Debugging Hooks

```bash
# Enable bash debug mode in your script
set -x

# Test a hook with a synthetic payload
echo '{"timestamp":1704614400000,"cwd":"/project","toolName":"bash","toolArgs":"{\"command\":\"sudo apt update\"}"}' \
  | .github/hooks/scripts/pre-tool-policy.sh
```

Troubleshooting table:

| Issue | Check |
|---|---|
| Hooks not executing | JSON valid? `version: 1` present? Script executable? Correct shebang? |
| Hooks timing out | Increase `timeoutSec`; optimize script |
| Invalid JSON output | Single line output; use `jq -c` |
| Script not found | Verify path is relative to `cwd` field |

---

## Lab 7: Full Hooks Pipeline (30 min)

**Goal:** Deploy a complete governance hooks setup for the calculator project.

**File structure to create:**

```
.github/
└── hooks/
    ├── policies.json
    ├── .gitignore       (contents: logs/)
    └── scripts/
        ├── session-banner.sh
        ├── log-prompt.sh
        └── pre-tool-policy.sh
```

**Steps:**

1. Create the `.github/hooks/` directory structure
2. Write `policies.json` referencing all three scripts for `sessionStart`, `userPromptSubmitted`, and `preToolUse`
3. Implement all three scripts using the examples above
4. Make scripts executable: `chmod +x .github/hooks/scripts/*.sh`
5. Start a Copilot session, verify the banner appears
6. Run a few prompts and check the audit log: `cat .github/hooks/logs/audit.jsonl | jq .`
7. Test a blocked command by asking Copilot: "Run sudo apt update"
8. Verify the deny appears in both the session and the audit log
9. Inspect denied events: `jq 'select(.event=="policyDeny")' .github/hooks/logs/audit.jsonl`

**Validation:**

- [ ] Banner visible on session start
- [ ] Audit entries written for each tool use
- [ ] `sudo` blocked with deny reason visible to Copilot

---

## References

- [About hooks](https://docs.github.com/en/copilot/concepts/agents/cloud-agent/about-hooks)
- [Hooks configuration reference](https://docs.github.com/en/copilot/reference/hooks-configuration)
- [Copilot CLI hooks tutorial](https://docs.github.com/en/copilot/tutorials/copilot-cli-hooks)

---

**Prev:** [Module 06: Automation](06-automation.md) | **Next:** [Module 08: Advanced and Certification Prep](08-advanced-and-certification.md)
