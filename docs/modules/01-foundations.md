# Module 01: Foundations

**Duration:** 90 minutes | [Course Home](../index.md) | **Next:** [Module 02: Core Interaction](02-core-interaction.md)

## Why This Module Exists

- Clear up the naming confusion before anything else
- Many developers have used `gh copilot suggest` and assume this course covers the same thing
- It does not
- This module establishes what the current product is and how to get it running

## Learning Objectives

- Explain the two-product history and why it matters for learners
- Install the standalone CLI on macOS, Linux, and Windows
- Authenticate via OAuth device flow and via environment variable PAT
- Understand the permission model and when each approval option is appropriate

---

## 1.1 Two Products, One Name (15 min)

### Timeline

- **2023:** GitHub Next technical preview introduced terminal command suggestion experiments, including `??`, `git?`, and `gh?` aliases. The focus was narrow: turn natural language into shell commands.
- **March 2024:** The `gh copilot` GitHub CLI extension reached GA. It was invoked as `gh copilot suggest "..."` and `gh copilot explain "..."`, plus the shell alias forms generated during setup. The experience remained command-oriented and shipped with only four commands.
- **October 25, 2025:** `gh copilot` was deprecated. The final release was v1.2.0, and the repository was archived on October 30, 2025.
- **2025-2026:** The standalone `copilot` binary replaced the extension. It shifted from command suggestion to a full interactive terminal coding assistant. Version `v1.0.44` was released on May 8, 2026.

### Why this history matters

- Learners may search for old blog posts and end up on deprecated `gh copilot` instructions
- The old product and the current product share a name, but not the same workflow
- This course is about the standalone `copilot` CLI, not the deprecated `gh copilot` extension

### Comparison

| Feature | `gh copilot` (deprecated) | `copilot` CLI (current) |
|---|---|---|
| Interface | `gh copilot suggest / explain` | Interactive terminal chat |
| Capability | Command suggestion only | Writes code, runs tests, creates PRs |
| Authentication | `gh` OAuth only | OAuth or fine-grained PAT |
| Model selection | Fixed | `/model`, Claude, GPT, Auto |
| MCP support | No | Yes |
| Free plan | Yes, limited | No, Pro required |
| Install | `gh extension install github/gh-copilot` | `npm install -g @github/copilot` |

---

## 1.2 Installation (30 min)

### Supported platforms

- Linux
- macOS
- Windows, PowerShell v6+ required

### Install methods

```bash
# macOS/Linux - install script (simplest)
curl -fsSL https://gh.io/copilot-install | bash

# macOS/Linux - Homebrew
brew install copilot-cli

# Windows - WinGet
winget install GitHub.Copilot

# All platforms - npm (Node.js 22+ required)
npm install -g @github/copilot

# Verify installation
copilot version
```

### Prerelease channel

```bash
npm install -g @github/copilot@prerelease
winget install GitHub.Copilot.Prerelease
brew install copilot-cli@prerelease
```

### Notes

- The npm install path requires Node.js 22 or later
- The install script is usually the fastest path on macOS and Linux
- Windows users need `pwsh`, not Windows PowerShell 5

---

## 1.3 Authentication (20 min)

### Token priority order

Copilot checks credentials in this sequence:

1. `COPILOT_GITHUB_TOKEN`
2. `GH_TOKEN` or `GITHUB_TOKEN`
3. OAuth token from the system keychain, created by a previous `/login`
4. `gh auth token` fallback

### Token types

| Token type | Prefix | Supported |
|---|---|---|
| OAuth device flow | `gho_` | Yes |
| Fine-grained PAT | `github_pat_` | Yes, needs `Copilot Requests` permission |
| GitHub App user-to-server | `ghu_` | Yes |
| Classic PAT | `ghp_` | **No** |

### Interactive auth

```bash
copilot
/login
```

### CI or automation auth

```bash
export COPILOT_GITHUB_TOKEN="github_pat_..."
copilot -p "Run tests"
```

### Credential storage

- macOS: Keychain Access
- Windows: Credential Manager
- Linux: `libsecret`, usually GNOME Keyring or KWallet
- Fallback: `~/.copilot/config.json`

---

## 1.4 The Trust and Permission Model (25 min)

### Approval options for tool execution

Every potentially dangerous tool execution prompts three choices:

1. **Yes** — allow this once, Copilot asks again next time
2. **Yes, and approve [TOOL] for the rest of the session** — broad approval for that tool pattern during the current session
3. **No** — deny the action, provide inline feedback, and let Copilot adjust

### Why session-wide approval needs care

- Approving a risky tool too broadly can remove the last safety checkpoint for the rest of the session
- Example: approving `shell(rm)` for the whole session could allow `rm -rf ./` later with no extra prompt
- Use session-wide approvals only for commands and tools you fully understand

### Key safety rules

- Copilot only reads and writes files in or below the directory it was launched from
- Never launch from `~/` or any directory that contains sensitive files you do not want modified
- `--allow-all` and `/yolo` grant full permissions, use them only in containers or disposable environments
- On first launch in a directory, Copilot asks whether to trust it; choose "remember" only for directories you always trust

### Key environment variable

```bash
export COPILOT_OFFLINE=true
```

`COPILOT_OFFLINE=true` disables telemetry and all GitHub network calls. Useful for BYOK mode and controlled environments.

---

## Lab 1: Setup and First Exploration (30 min)

**Goal:** Get the CLI running and use it to understand a codebase.

**Steps:**

1. Install the CLI using the method appropriate for your OS
2. Run `copilot version` to confirm the installation
3. Authenticate with `/login`
4. Navigate to a GitHub repository you work with regularly
5. Start a session and ask: `Give me an overview of this project's architecture`
6. Ask a follow-up: `What are the main entry points and how does the code flow from a user request to a response?`
7. Use `!git log --oneline -10` to run a git command directly without involving the LLM
8. Type `/usage` to see token consumption so far
9. Exit with `/exit`

**Validation:**

- [ ] CLI installed and `copilot version` succeeds
- [ ] Authentication completed successfully
- [ ] One codebase exploration session completed

**Common issues:**

- `Classic PAT not supported`: generate a fine-grained PAT with `Copilot Requests` permission instead
- CLI not found after npm install: ensure your global npm bin directory is in `PATH`
- Windows issue: requires PowerShell 6+ with `pwsh`, not Windows PowerShell 5

---

## References

- [Copilot CLI repository](https://github.com/github/copilot-cli)
- [Install guide](https://docs.github.com/en/copilot/how-tos/copilot-cli/set-up-copilot-cli/install-copilot-cli)
- [Authentication guide](https://docs.github.com/en/copilot/how-tos/copilot-cli/set-up-copilot-cli/authenticate-copilot-cli)

---

**Next:** [Module 02: Core Interaction](02-core-interaction.md)
