# Module 08: Advanced Patterns and Certification Prep

**Duration:** 90 minutes | **Prev:** [Module 07: Hooks and Governance](07-hooks-and-governance.md) | [Course Home](../index.md)

## Learning Objectives

- Apply the Explore → Plan → Code → Commit workflow to complex multi-file changes
- Use `/fleet` to run parallel agent tasks and `Ctrl+X b` to manage background work
- Work across multiple repositories in a single session
- Map all course content to the 7 GitHub Copilot Certification exam domains

---

## 8.1 The Explore → Plan → Code → Commit Workflow (25 min)

The canonical pattern for complex changes. Used when a task touches multiple files, has unclear scope, or requires architectural decisions.

```text
Read the authentication files but don't write code yet        # Explore
/plan Implement password reset flow                           # Plan
# Review plan.md, Ctrl+y to open in editor, edit checkboxes
# Answer any clarifying questions Copilot asks
Proceed with the plan                                         # Implement
Run the tests and fix any failures                            # Verify
Commit these changes with a descriptive message               # Commit
```

### When to Use Each Phase

| Phase | Command/action | Purpose |
|---|---|---|
| Explore | "Read X but don't write code" | Understand before acting |
| Plan | `/plan DESCRIPTION` or `Shift+Tab` | Structure work, get alignment |
| Implement | "Proceed with the plan" | Execute with checkpoints |
| Verify | "Run tests" or `!npm test` | Validate changes |
| Commit | "Commit these changes" | Clean history |

### Plan Mode Decision Guide

| Use `/plan` | Skip it |
|---|---|
| Multi-file feature implementation | One-line bug fix |
| Refactoring with many touch points | Single-file change |
| Large-scale migration or upgrade | Quick exploratory reading |
| Onboarding to an unfamiliar codebase | When you know exactly what to build |

### Migration Checklist Pattern

For large migrations, use a self-updating checklist:

```text
Run the TypeScript compiler and save all type errors as a checklist in migration-checklist.md.
Then fix each error one by one, checking each item off as you go.
```

Copilot writes the checklist, then works through it systematically, checking off items after each fix.

---

## 8.2 Multi-Repo Workflows (20 min)

### Launch from Parent Directory

```bash
cd ~/projects && copilot
# Copilot can access all subdirectories
```

### Add Directories Mid-Session

```bash
/add-dir /Users/me/projects/auth-service
/add-dir /Users/me/projects/frontend
/list-dirs     # verify what's accessible
```

### Cross-Repo Prompt Pattern

Use `@` references to call out specific repos explicitly:

```text
I need to update the user authentication API.
The changes span three repos:
- @/Users/me/projects/api-gateway (routing changes needed)
- @/Users/me/projects/auth-service (core JWT logic)
- @/Users/me/projects/frontend (update the login component)

Start by showing me the current authentication flow across all three.
Don't write any code yet.
```

Then proceed step by step, one repo at a time, or let Copilot sequence the changes.

### Multi-Repo PR Strategy

For changes spanning repos, use `/delegate` for the secondary repos while working on the primary one locally:

```text
/delegate Update the frontend login component to use the new /auth/v2 endpoint
& Update the API gateway routing config for the new auth routes
# Then implement the auth-service changes locally
```

---

## 8.3 `/fleet`: Parallel Agent Orchestration (20 min)

Fleet mode breaks a task into subtasks and runs them as parallel subagents.

```bash
/fleet Migrate all class components to functional React components with hooks
```

Copilot analyzes the scope, identifies independent subtasks (one per component), launches parallel agents, and merges results.

### Managing Fleet Agents

```bash
/tasks              # list running agents and their status
Ctrl+X b            # promote the current running task to background (detach without stopping)
Ctrl+Z              # suspend current task to background (Unix)
```

### When to Use `/fleet`

| Good fit | Poor fit |
|---|---|
| Large-scale refactoring (many independent files) | Tasks with strict ordering dependencies |
| Generating tests for multiple modules in parallel | Tasks requiring shared state between agents |
| Running independent feature implementations | Debugging that needs step-by-step inspection |
| Migrating patterns across a large codebase | Single-file changes |

### `/fleet` vs. `/delegate`

| | `/fleet` | `/delegate` |
|---|---|---|
| Where it runs | Local machine (parallel local agents) | GitHub cloud agent |
| Machine required? | Yes | No |
| PR creation | No (you create the PR after) | Yes (creates draft PR automatically) |
| Use for | Large local tasks | Async tasks you want off your machine |

---

## 8.4 The Research Agent (10 min)

The built-in research agent uses an orchestrator/subagent model. Unlike a standard prompt, it dispatches multiple parallel subagents to gather information, then synthesizes a cited report.

```bash
/research What are best practices for implementing OAuth2 PKCE in Node.js?
/research Compare database options for a multi-tenant SaaS: PostgreSQL vs. CockroachDB vs. PlanetScale
/research What patterns does this codebase use for error handling?
```

Output format: structured report with inline citations and a footnotes section. Useful for architecture decisions, library evaluations, and technology choices.

---

## 8.5 Productivity Metrics and Key UX Patterns (10 min)

GitHub-recommended metrics to track the impact of Copilot CLI:

- Time from issue assignment to pull request opened
- Number of review iteration cycles before merge
- Code review feedback turnaround
- Test coverage percentage over time

Highest-impact UX habits:

- `copilot --continue` as your session resume command (muscle memory)
- `/delegate` for any tangential task that would distract from current focus
- Auto-compaction at 95% context = never manually manage context
- `Ctrl+T` to inspect model reasoning when Copilot seems to be going in the wrong direction
- `Ctrl+X b` to move a long-running task (for example, a large test suite) to background while prompting

---

## 8.6 GitHub Copilot Certification Alignment (30 min)

The GitHub Copilot Certification exam covers 7 domains. Every domain is addressed in this course:

| Certification Domain | Covered In | Key topics |
|---|---|---|
| **Responsible AI** | Module 1 (trust model), Module 7 (hooks, governance) | Permission model, audit logging, human oversight |
| **Copilot plans and features** | Module 1 (product history, subscription tiers) | Free vs Pro vs Enterprise, legacy vs current CLI |
| **Copilot data and functionality** | Module 1 (auth, telemetry), Module 7 (privacy) | Token types, COPILOT_OFFLINE, hooks redaction |
| **Prompt engineering** | Modules 2-5 (@, #, instructions, agents) | Context injection, instructions hierarchy, agent personas |
| **AI developer use cases** | Modules 3, 6 (PR workflows, CI/CD) | Issue→PR pipeline, /pr auto, GitHub Actions |
| **Testing with Copilot** | Module 3 (test gen in lab), Module 6 (CI) | Unit test generation, /pr fix ci, test-driven workflow |
| **Privacy and exclusions** | Module 4 (instructions), Module 7 (hooks) | copilot-instructions.md, token redaction in hook logs |

### Exam Preparation Checklist

Practical knowledge to confirm before sitting the exam:

- [ ] Can explain the difference between `gh copilot` (deprecated) and standalone `copilot` CLI
- [ ] Can describe all 3 interaction modes and when to use each
- [ ] Understands the custom instructions hierarchy (global vs repo vs path-specific)
- [ ] Can describe the full Issue → Branch → Code → PR workflow
- [ ] Understands `/pr auto` vs `/delegate` trade-offs
- [ ] Can explain what hooks are and which hook type can block execution
- [ ] Can describe the `preToolUse` deny output format
- [ ] Understands token types and why classic PATs (`ghp_`) are not supported
- [ ] Can articulate Copilot CLI's trust/permission model and why it matters
- [ ] Knows how to use Copilot CLI in CI (env var auth, `--no-ask-user`, `--deny-tool`)

---

## Capstone Lab: Full End-to-End Workflow (30 min)

**Goal:** Demonstrate mastery across all 8 modules using the calculator project.

**Steps:**

1. **Issue creation:** "Create a feature request issue for adding a scientific calculator mode (sin, cos, tan, sqrt)"
2. **Planning:** Switch to Plan mode (`/plan Add scientific calculator mode`), answer clarifying questions, review and approve the plan
3. **Parallel implementation:** Use `/fleet` to implement the math functions (sin/cos/tan) in parallel subagents, one per function
4. **Testing:** Ask Copilot to generate comprehensive tests for the new functions
5. **PR workflow:** Use `/pr auto` to create the PR, fix any CI failures, and address review feedback automatically
6. **Delegation:** While `/pr auto` runs, `/delegate` a documentation update: "Add a usage section to the README showing all scientific functions with examples"
7. **Merge:** Merge the PR and verify both the feature PR and delegated documentation PR are complete
8. **Export:** `/share gist` to save the full session as a GitHub Gist

**Validation:**

- [ ] Merged PR with tests passing
- [ ] Documentation updated via `/delegate`
- [ ] Session exported as a Gist

---

**Course complete.** Return to the [Course Home](../index.md) for the full module list and resources.

## References

- [CLI best practices](https://docs.github.com/en/copilot/how-tos/copilot-cli/cli-best-practices)
- [CLI command reference](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference)
- [GitHub Copilot Certification](https://learn.github.com/credentials)
- [About GitHub Certifications](https://docs.github.com/en/get-started/showcase-your-expertise-with-github-certifications/about-github-certifications)

---

**Prev:** [Module 07: Hooks and Governance](07-hooks-and-governance.md) | [Course Home](../index.md)
