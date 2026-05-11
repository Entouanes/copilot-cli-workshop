# GitHub Copilot CLI: Quick Reference

[Course Home](../index.md) | [Lab Project Guide](lab-guide.md)

---

## 1. Installation and Authentication

```bash
# Install
npm install -g @github/copilot   # requires Node.js 22+
brew install copilot-cli
winget install GitHub.Copilot
curl -fsSL https://gh.io/copilot-install | bash

# Auth
copilot → /login                 # OAuth device flow
export COPILOT_GITHUB_TOKEN="github_pat_..."  # env var

# Token types supported
# gho_         (OAuth)
# github_pat_  (fine-grained PAT — needs "Copilot Requests" permission)
# ghu_         (GitHub App)
# NOT supported: ghp_ (classic PAT)
```

---

## 2. Start Modes

```bash
copilot                         # interactive
copilot -p "PROMPT"             # single prompt, exits
copilot -sp "PROMPT"            # silent (script-friendly)
copilot -i "PROMPT"             # interactive with opening prompt
copilot --continue              # resume last session
copilot -C /path/to/repo -p ""  # different working directory
copilot --plan                  # start in plan mode
copilot --autopilot             # start in autopilot mode
copilot --mode=autopilot        # same as above
```

---

## 3. Context Injection Operators

```text
@FILENAME          include file contents in context
#ISSUE_NUMBER      include a GitHub issue or PR
!COMMAND           run shell command without LLM
```

---

## 4. Mode Cycling

```text
Shift+Tab          cycle: Standard → Plan → Autopilot → Standard
/plan DESCRIPTION  enter plan mode with a specific task
```

---

## 5. Keyboard Shortcuts

| Shortcut | Action |
|---|---|
| `Shift+Tab` | Cycle Standard/Plan/Autopilot |
| `Ctrl+G` | Edit prompt in `$EDITOR` |
| `Ctrl+R` | Reverse command history search |
| `Ctrl+V` | Paste clipboard as attachment |
| `Ctrl+Enter` | Queue message while agent busy |
| `Ctrl+X /` | Run slash command mid-prompt |
| `Ctrl+X b` | Move task to background |
| `Ctrl+T` | Toggle reasoning display |
| `Ctrl+F` | Search timeline |
| `Ctrl+L` | Clear screen |
| `Ctrl+C` | Cancel / clear input |
| `Ctrl+D` | Shutdown |
| `double-Esc` | Cancel in-flight work |
| `Shift+Enter` | Insert newline |
| `Page Up/Down` | Scroll timeline |
| `/undo` | Revert last turn + file changes |

---

## 6. Slash Commands Reference

| Command | Purpose |
|---|---|
| `/model` | Select AI model |
| `/diff` | Review changes in current directory |
| `/pr [view\|create\|fix\|auto]` | Manage pull requests |
| `/delegate PROMPT` or `& PROMPT` | Hand off to cloud agent |
| `/review` | Code review agent |
| `/research TOPIC` | Deep research mode |
| `/plan PROMPT` | Create implementation plan |
| `/lsp [show\|test\|reload]` | Language server management |
| `/mcp [show\|add\|edit\|delete\|reload]` | MCP server management |
| `/fleet` | Parallel multi-agent mode |
| `/tasks` | View running agents |
| `/instructions` | View/toggle instructions |
| `/env` | Show loaded context |
| `/skills list` | List available skills |
| `/agent` | Browse and select agents |
| `/session` | Current session info |
| `/compact` | Compress history |
| `/context` | Token usage visualization |
| `/usage` | Session usage metrics |
| `/share gist` | Export to GitHub Gist |
| `/share file PATH` | Export to Markdown file |
| `/rename NAME` | Rename session |
| `/resume ID` | Switch session |
| `/clear` or `/new` | Fresh conversation |
| `/changelog` | CLI changelog |
| `/experimental on/off` | Toggle experimental features |
| `/help` | Help |
| `/exit` | Exit |
| `/keep-alive` | Prevent system sleep |

---

## 7. `/pr` Subcommands

```bash
/pr                  # view PR status
/pr view web         # open in browser
/pr create           # create or update PR
/pr fix feedback     # address review comments
/pr fix conflicts    # sync with base branch
/pr fix ci           # fix failing CI (loops until green)
/pr fix all          # run all three fix phases
/pr auto             # create + fix everything automatically
```

---

## 8. Tool Permission Flags

```bash
# Allow patterns
--allow-all-tools
--allow-tool='shell(git:*)'
--allow-tool='shell(npm run:*)'
--allow-tool='write'
--allow-all-paths
--allow-all-urls

# Deny patterns (override allow)
--deny-tool='shell(rm)'
--deny-tool='shell(git push)'
--deny-tool='shell(sudo)'

# Safe CI pattern
copilot --allow-all-tools \
  --deny-tool='shell(rm)' \
  --deny-tool='shell(git push)' \
  --no-ask-user \
  -p "TASK"
```

---

## 9. Environment Variables

```bash
COPILOT_GITHUB_TOKEN   # auth token (highest priority)
GH_TOKEN               # auth fallback
GITHUB_TOKEN           # auth fallback
COPILOT_HOME           # config directory (default: ~/.copilot/)
COPILOT_OFFLINE=true   # disable telemetry and GitHub network
COPILOT_GH_HOST        # GitHub Enterprise hostname
GITHUB_COPILOT_PROMPT_MODE_REPO_HOOKS=true   # enable hooks in -p mode
GITHUB_COPILOT_PROMPT_MODE_WORKSPACE_MCP=true # enable MCP in -p mode
```

---

## 10. Config File Locations

```text
~/.copilot/config.json              # auth and general config
~/.copilot/mcp-config.json          # MCP server definitions
~/.copilot/lsp-config.json          # LSP server definitions
~/.copilot/copilot-instructions.md  # global custom instructions
~/.copilot/agents/                  # user-level custom agents
~/.copilot/skills/                  # user-level skills
~/.copilot/session-state/           # session history

.github/copilot-instructions.md     # repo custom instructions
.github/instructions/**/*.instructions.md  # path-specific instructions
.github/agents/                     # repo-level custom agents
.github/hooks/*.json                # hook configurations
.github/lsp.json                    # repo-level LSP config
.github/copilot/settings.json       # merge strategy etc.
AGENTS.md                           # also used by cloud agent
```

---

## 11. Hook Types

| Hook | Fires | Can deny? |
|---|---|---|
| `sessionStart` | Session begins | No |
| `sessionEnd` | Session ends | No |
| `userPromptSubmitted` | Prompt submitted | No |
| `preToolUse` | Before tool call | **Yes** |
| `postToolUse` | After tool completes | No |
| `agentStop` | Agent finishes responding | No |
| `subagentStop` | Subagent completes | No |
| `errorOccurred` | Error occurs | No |

`preToolUse` deny output:

```json
{ "permissionDecision": "deny", "permissionDecisionReason": "Reason" }
```

---

## 12. Authentication Priority Order

```
1. COPILOT_GITHUB_TOKEN (env var)
2. GH_TOKEN or GITHUB_TOKEN (env var)
3. OAuth keychain token (from /login)
4. gh auth token (gh CLI fallback)
```

---

[Course Home](../index.md) | [Lab Project Guide](lab-guide.md)
