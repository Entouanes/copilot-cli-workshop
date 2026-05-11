# Module 02: Core Interaction

**Duration:** 120 minutes | **Prev:** [Module 01: Foundations](01-foundations.md) | **Next:** [Module 03: GitHub Integration](03-github-integration.md)

## Learning Objectives

- Start sessions using interactive, programmatic, and resume modes
- Cycle between Standard, Plan, and Autopilot modes with Shift+Tab
- Use keyboard shortcuts to accelerate common workflows
- Manage sessions: name, resume, compact, share, and understand the token context

---

## 2.1 Starting a Session (20 min)

All start modes:

```bash
copilot                     # interactive (default)
copilot -p "PROMPT"         # single prompt, exits after
copilot -sp "PROMPT"        # single prompt, silent (suppresses usage info, for scripts)
copilot -i "PROMPT"         # interactive with auto-executed opening prompt
copilot --continue          # resume most recent session in current directory
copilot -C /path/to/repo    # start in a different directory (v1.0.42+)
copilot --name "my-session" # start a named session
```

Context injection operators, use inline in any prompt:

- `@FILENAME` — paste file contents into context, tab-completion available
- `#ISSUE_NUMBER` — include a GitHub issue or PR by number
- `!COMMAND` — execute shell command without involving the LLM

Examples:

```text
Explain @src/auth/middleware.ts
What's the status of PR #234?
!git log --oneline -5
```

---

## 2.2 The Three Modes (30 min)

Toggle with **Shift+Tab** to cycle: Standard → Plan → Autopilot → Standard

Or set at startup:

```bash
copilot --mode=plan
copilot --mode=autopilot
copilot --plan
copilot --autopilot
```

### Standard mode (default)

- Conversational
- Each prompt executes immediately
- Best for: exploration, quick tasks, debugging, single-step changes

### Plan mode

- Copilot reads the codebase
- Asks clarifying questions
- Writes a structured `plan.md` with checkboxes
- Waits for your approval before writing any code

Session plan stored at: `~/.copilot/session-state/{session-id}/plan.md`

```text
/plan Add OAuth2 authentication with Google and GitHub providers
```

Flow:

- Copilot asks questions such as: "Should I support refresh tokens?"
- You answer
- Plan is generated
- You review it
- Implementation begins after approval

Use `Ctrl+y` to open the plan in your default editor before approving.

When to use Plan mode:

| Use it | Skip it |
|---|---|
| Multi-file feature implementation | One-line bug fix |
| Refactoring with many touch points | Exploratory reading |
| Large-scale migration | Single file change |
| When you want to review before any code is written | When you trust the task is simple |

### Autopilot mode

- Runs autonomously without interruptions
- Default limit: 5 auto-continues before pausing for confirmation

Control via:

```bash
copilot --autopilot
copilot --max-autopilot-continues=10
```

Best for: batch work, refactoring, delegated tasks. Only use in directories you fully trust with `--allow-all-tools`.

---

## 2.3 Keyboard Shortcuts (25 min)

| Shortcut | Action |
|---|---|
| `Shift+Tab` | Cycle Standard / Plan / Autopilot |
| `Ctrl+G` or `Ctrl+X e` | Edit current prompt in `$EDITOR` |
| `Ctrl+R` | Reverse search command history |
| `Ctrl+V` | Paste clipboard content as a file attachment |
| `Ctrl+Enter` or `Ctrl+Q` | Queue a message while the agent is busy |
| `Ctrl+X /` | Run a slash command mid-prompt |
| `Ctrl+X b` | Move current running task to background |
| `Ctrl+T` | Toggle reasoning/thinking display |
| `Ctrl+F` | Search the session timeline |
| `Ctrl+L` | Clear the screen |
| `Ctrl+C` | Cancel operation / clear input |
| `Ctrl+D` | Shutdown |
| `double-Esc` | Cancel in-flight work |
| `Shift+Enter` | Insert newline in input (multi-line prompt) |
| `Page Up/Down` | Scroll timeline |
| `/undo` or `/rewind` | Revert last turn and its file changes |

Practical tips:

- Use `Ctrl+V` to paste a screenshot; Copilot can reason about images
- `Ctrl+T` is useful when Copilot seems to be thinking too long; inspect the reasoning path
- Use `Ctrl+X b` to offload a running task (for example, a long test run) so you can keep chatting

---

## 2.4 Session Management (25 min)

Session storage layout:

```text
~/.copilot/session-state/{session-id}/
├── events.jsonl       # complete conversation history
├── workspace.yaml     # session metadata
├── plan.md            # implementation plan (if created)
├── checkpoints/       # auto-compaction history
└── files/             # persistent session artifacts
```

Key session commands:

| Command | Purpose |
|---|---|
| `/session` | Info about current session |
| `/session checkpoints` | List all checkpoints |
| `/session files` | List files touched in this session |
| `/session plan` | View current plan |
| `/compact` | Manually compress history to save tokens |
| `/context` | Visualize token usage against context window |
| `/usage` | Show session usage metrics |
| `/share gist` | Export session to a secret GitHub Gist |
| `/share file PATH` | Export to local Markdown file |
| `/share html PATH` | Export as interactive HTML |
| `/rename My-session` | Rename the current session |
| `/resume SESSION-ID` | Switch to a different session |
| `/clear` or `/new` | Start a fresh conversation |

Auto-compaction:

- Triggers automatically at 95% context utilization
- Enables effectively unlimited session length

Resume pattern:

```bash
copilot --continue
copilot --resume=auth-refactor
```

---

## 2.5 Curated Slash Commands Reference (20 min)

From the 60+ available slash commands, the most important:

| Command | Purpose |
|---|---|
| `/model` or `/models` | Select AI model |
| `/diff` | Review all changes in the current directory |
| `/pr [view\|create\|fix\|auto]` | Manage pull requests, see [Module 03](03-github-integration.md) |
| `/delegate PROMPT` | Hand off task to Copilot cloud agent |
| `/review` | Run code review agent |
| `/research TOPIC` | Deep research with orchestrator/subagent model |
| `/lsp [show\|test\|reload]` | Manage language server config |
| `/mcp [show\|add\|edit\|delete\|reload]` | Manage MCP servers |
| `/fleet` | Enable parallel multi-agent execution |
| `/instructions` | View or toggle active instruction files |
| `/env` | Show loaded instructions, MCP, skills, agents |
| `/plan PROMPT` | Create implementation plan |
| `/experimental [on\|off]` | Toggle experimental features |
| `/changelog` | Show CLI changelog |
| `/keep-alive` | Prevent system sleep |
| `/help` | Show help |

Model selection guidance:

| Model | Best for | Trade-off |
|---|---|---|
| **Auto** | Reduced rate limiting, lower latency | Smart routing between models |
| **Claude Sonnet 4.5** (default) | Day-to-day coding, routine tasks | Fast, cost-effective |
| **Claude Opus 4.5** | Complex architecture, hard debugging | Most capable, more premium requests |
| **GPT-5.2 Codex** | Code generation, code review | Strong for reviewing output from other models |

---

## Lab 2: Modes and Session Mastery (30 min)

**Goal:** Practice all three modes and session management on the course project.

**Setup:** Clone or use a GitHub repository with some existing code. The calculator project from the [Lab Project Guide](../resources/lab-guide.md) is ideal.

**Steps:**

1. Start a session in Standard mode: `copilot --continue` (or a fresh session)
2. Ask: "What does this project do and where are the main files?"
3. Include a file in context: "Explain @src/calculator.js in detail"
4. Check token usage: `/context`
5. Switch to Plan mode with `Shift+Tab`
6. Run: `/plan Add a percentage calculation feature`
7. Answer Copilot's clarifying questions, then review the plan
8. Press `Ctrl+y` to inspect or edit the plan in your editor
9. Approve the plan
10. Let Copilot implement one step, then use `/undo` to revert it
11. Switch to Autopilot mode with `Shift+Tab` again
12. Run: "Implement the percentage feature as planned"
13. After completion, run `/diff` to review all changes
14. Export the session: `/share file ./session-notes.md`
15. Start a fresh session with `/new`, then resume with `copilot --continue`

**Validation:**

- [ ] You can switch modes with Shift+Tab
- [ ] You inspected context with `/context`
- [ ] You reverted a change with `/undo`
- [ ] You exported and resumed a session

---

## References

- [CLI command reference](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference)
- [CLI best practices](https://docs.github.com/en/copilot/how-tos/copilot-cli/cli-best-practices)

---

**Prev:** [Module 01: Foundations](01-foundations.md) | **Next:** [Module 03: GitHub Integration](03-github-integration.md)
