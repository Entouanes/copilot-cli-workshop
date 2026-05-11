# GitHub Copilot CLI Workshop

**Audience:** Developers, DevOps engineers, platform teams, and technical leads adopting GitHub Copilot CLI
**Duration:** ~12-15 hours across 8 modules
**Prerequisites:** Git basics, terminal comfort, GitHub account with Copilot access, a practice repository

---

!!! warning "Critical: Two products share the same name"
    | Product | Status | What it does |
    |---|---|---|
    | `gh copilot` | Deprecated Oct 2025 (v1.2.0, archived Oct 30, 2025) | GitHub CLI extension for `suggest` and `explain` only |
    | `copilot` | Current standalone CLI (v1.0.44, May 2026) | Full agentic CLI for code changes, tests, GitHub workflows, automation |

    **This workshop is about the standalone `copilot` binary.** Any material built around `gh copilot suggest` or `gh copilot explain` is historical context only.

---

## Learning Path

| Module | Title | Duration | Lab Deliverable |
|---|---|---|---|
| [01](modules/01-foundations.md) | Foundations | 90 min | Installed CLI, authenticated session, repository walkthrough |
| [02](modules/02-core-interaction.md) | Core Interaction | 120 min | Saved plan created from an interactive session |
| [03](modules/03-github-integration.md) | GitHub Integration | 120 min | End-to-end issue to PR workflow completed |
| [04](modules/04-customization.md) | Customization | 90 min | Project instruction file and custom agent created |
| [05](modules/05-mcp-and-lsp.md) | MCP and LSP | 90 min | MCP-enabled workflow using external tools and code intelligence |
| [06](modules/06-automation.md) | Automation | 120 min | GitHub Actions workflow calling Copilot CLI |
| [07](modules/07-hooks-and-governance.md) | Hooks and Governance | 120 min | Hook pipeline with audit logging and policy checks |
| [08](modules/08-advanced-and-certification.md) | Advanced and Certification Prep | 90 min | Capstone workflow using `/plan`, `/fleet`, `/pr`, and `/delegate` |

---

## How to Use This Workshop

- **Start at Module 01** — it clears up the naming confusion and gets the CLI running
- **Build fluency in Module 02** — covers the core interaction model, sessions, shortcuts, and slash commands
- **Ship real work in Module 03** — issues, branches, pull requests, and GitHub-native workflows
- **Adapt the assistant in Module 04** — instructions, agents, and skills for team-specific behavior
- **Extend capability in Module 05** — MCP servers and language intelligence for broader tool access
- **Automate at scale in Module 06** — programmatic use, CI, and GitHub Actions
- **Control risk in Module 07** — hooks, policy enforcement, and enterprise rollout patterns
- **Finish strong in Module 08** — advanced workflows and GitHub Copilot Certification prep

Each module builds on the previous one using a single continuous Node.js calculator project. See the [Lab Project Guide](resources/lab-guide.md) for setup instructions.

---

## Prerequisites Checklist

- [ ] Git installed, comfortable with branch / commit / PR workflow
- [ ] Terminal access on Windows (PowerShell 6+), macOS, or Linux
- [ ] GitHub account with GitHub Copilot access (Pro, Pro+, Business, or Enterprise)
- [ ] A practice repository available for labs
- [ ] Permission to create issues, branches, and pull requests in that repository
- [ ] Node.js 22+ available (required for npm install path)

---

## Resources

- [Quick Reference Cheatsheet](resources/cheatsheet.md) — commands, shortcuts, slash commands at a glance
- [Lab Project Guide](resources/lab-guide.md) — the Node.js calculator project used across all labs

---

## Key External Links

- [Copilot CLI repository](https://github.com/github/copilot-cli)
- [Install guide](https://docs.github.com/en/copilot/how-tos/copilot-cli/set-up-copilot-cli/install-copilot-cli)
- [CLI command reference](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference)
- [CLI best practices](https://docs.github.com/en/copilot/how-tos/copilot-cli/cli-best-practices)
- [Hooks tutorial](https://docs.github.com/en/copilot/tutorials/copilot-cli-hooks)
- [Official GitHub Skills exercise](https://github.com/skills/create-applications-with-the-copilot-cli)
- [GitHub Copilot Certification](https://learn.github.com/credentials)
- [Microsoft Learn Copilot path](https://learn.microsoft.com/en-us/training/paths/copilot/)

---

## Certification Alignment

This workshop maps directly to the GitHub Copilot Certification skill domains: product capabilities, prompt engineering, AI developer workflows, testing, privacy, and responsible use. Use [Module 08](modules/08-advanced-and-certification.md) as the final review pass.
