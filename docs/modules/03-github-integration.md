# Module 03: GitHub Integration

**Duration:** 120 minutes | **Prev:** [Module 02: Core Interaction](02-core-interaction.md) | **Next:** [Module 04: Customization](04-customization.md)

## Learning Objectives

- Use the built-in GitHub MCP server to interact with GitHub.com via natural language
- Execute the full Issue → Branch → Code → PR workflow
- Apply all `/pr` subcommands, including the automated fix loops
- Choose between local `/pr` workflows and remote `/delegate` based on the task

---

## 3.1 The Built-in GitHub MCP Server (20 min)

Copilot CLI ships with a built-in GitHub MCP server that exposes GitHub API tools.

Start with full GitHub toolset:

```bash
copilot --enable-all-github-mcp-tools
```

Add specific toolsets only:

```bash
copilot --add-github-mcp-toolset=TOOLSET
```

Disable the built-in server entirely, if using your own MCP config:

```bash
copilot --disable-builtin-mcps
```

What you can do via natural language with GitHub MCP enabled:

- `List my open PRs`
- `List all open issues assigned to me in owner/repo`
- `Find good first issues for a new contributor in octo-org/octo-repo`
- `I've been assigned issue #1234. Start working on it in a suitable branch.`
- `Raise an improvement issue: in src/app.py the file handle is never closed`
- `Check the changes made in PR #57575`
- `List any Actions workflows in this repo that add comments to PRs`
- `Merge all open PRs I've created in octo-org/octo-repo`
- `Create a PR that updates the README, changing 'How to run' to 'Example usage'`

Context shortcuts in session:

- `@FILENAME` — include file contents
- `#NUMBER` — include a GitHub issue or PR by number

---

## 3.2 The Full Issue → PR Workflow (40 min)

The canonical end-to-end workflow, from the official [GitHub Skills exercise](https://github.com/skills/create-applications-with-the-copilot-cli):

### Step 1: Create an Issue

```text
Create a GitHub issue for a Node.js calculator app feature
requesting addition, subtraction, multiplication, and division.
Use the .github/ISSUE_TEMPLATE/feature_request.md template.
Create the issue in this owner/repository using gh CLI commands.
List the issue link when complete.
```

### Step 2: Create a Branch and Write Code

```bash
# Start with all permissions enabled
copilot --allow-all --enable-all-github-mcp-tools
```

```text
Create and push a new branch called 'create-calc-app'

Now implement a Node.js CLI calculator in src/calculator.js with
functions for addition, subtraction, multiplication, and division.
Comment each function with its operation.
```

### Step 3: Write Tests

```text
Create comprehensive unit tests in src/tests/calculator.test.js
covering all four operations and edge cases like division by zero.
Use Jest or a similar popular testing framework.
Make sure all tests pass.
```

### Step 4: Commit and Push

```text
Add all calculator and test files to git.
Commit with message "Implement basic calculator operations and tests"
Push the changes.
```

### Step 5: Create PR with Copilot as Reviewer

```text
Create a pull request from the current branch.
Title: "Add calculator enhancements"
Add @copilot as a reviewer and request a review.
Link the PR to the issue we created so it closes automatically on merge.
List the PR link when complete.
```

### Step 6: Merge and Verify

```text
Merge the pull request.
Verify the linked issue is now closed.
```

!!! note
    Copilot creates PRs on your behalf but **you are marked as the PR author**.

---

## 3.3 The `/pr` Command (30 min)

All subcommands operate on the **current branch**. They require an existing branch with commits.

| Subcommand | Action | Commits/pushes? | Requires existing PR? |
|---|---|---|---|
| `/pr` or `/pr view` | Show PR status | No | Yes |
| `/pr view web` | Open PR in browser | No | Yes |
| `/pr create` | Create or update PR, follows PR template | Yes | No |
| `/pr fix feedback` | Address review comments | Yes | Yes |
| `/pr fix conflicts` | Sync with base branch, resolve conflicts | Yes | Yes |
| `/pr fix ci` | Diagnose and fix failing CI, loops until green | Yes | Yes |
| `/pr fix` or `/pr fix all` | Run all three fix phases in order | Yes | Yes |
| `/pr auto` | Create PR if needed, then loop all fixes until fully green | Yes | No |

Usage examples:

```bash
/pr create
/pr create prefix the PR title 'Team X: '
/pr fix feedback
/pr fix ci focus on test failures
/pr auto
/pr auto include migration notes in the description
```

`/pr fix ci` behavior: Copilot diagnoses the failure, makes fixes, pushes, re-checks CI, and repeats until all checks pass or it determines further progress is not possible.

`/pr auto` behavior: creates PR if none exists, then loops through feedback → conflicts → CI phases until everything is green.

Configure merge strategy in `.github/copilot/settings.json` or `~/.copilot/settings.json`:

```json
{ "mergeStrategy": "rebase" }
```

---

## 3.4 `/delegate`: Async Cloud Agent (20 min)

```bash
/delegate Implement user authentication with JWT tokens

# Shorthand: & prefix
& Fix the memory leak in the connection pool
```

What happens when you run `/delegate`:

1. Copilot commits any unstaged changes as a checkpoint
2. Creates a new remote branch automatically
3. Opens a draft pull request on GitHub.com
4. Copilot cloud agent works autonomously in the background
5. Streams progress output to your terminal
6. Requests your review when complete

You can close your machine; the work continues in the cloud.

**`/delegate` vs. local `/pr` workflow:**

| Dimension | `/delegate` | Local `/pr` workflow |
|---|---|---|
| Where code runs | GitHub cloud agent, remote | Your machine, local |
| Branch creation | Automatic | You create it |
| PR state | Starts as draft | Regular PR |
| Machine required? | No, can shut down | Yes |
| Premium requests | Yes | No |
| Use when | Tangential or async work | Core feature you'll review closely |

Common `/delegate` use cases:

- Documentation updates
- Refactoring unrelated modules
- Adding tests for existing code
- Fixing lint/style issues across the codebase
- Dependency upgrades

---

## 3.5 Remote Session Control (10 min)

Control a running Copilot CLI session from GitHub.com or GitHub Mobile:

```bash
copilot --remote        # enable remote access at startup
/remote on              # enable mid-session
/remote off             # disable
```

Sessions appear under "Recent agent sessions → Copilot" on GitHub.com and in the repo's Agents tab. Useful for: checking progress from your phone, adding context from the web UI while Copilot works locally.

---

## Lab 3: Full Feature Lifecycle (30 min)

**Goal:** Build a complete feature from issue to merged PR.

**Project:** Use the calculator project from the [Lab Project Guide](../resources/lab-guide.md).

**Steps:**

1. Start a session: `copilot --allow-all --enable-all-github-mcp-tools`
2. Create a feature request issue for adding a "history" feature (log the last 10 calculations)
3. Create a branch called `feature/history`
4. Implement the history feature in `src/calculator.js`
5. Write unit tests in `src/tests/calculator.test.js` covering the history feature
6. Commit and push
7. Create a PR with `@copilot` as reviewer: `/pr create`
8. Wait for Copilot's review, then address the feedback: `/pr fix feedback`
9. Check CI, if any: `/pr fix ci`
10. Use `/pr auto` on a second feature to see the full loop
11. Merge: ask Copilot to merge the PR
12. Verify: "List the closed issues in this repository"
13. Bonus: `/delegate` a documentation update for the README while moving on

**Validation:**

- [ ] Merged PR with linked issue
- [ ] Copilot review addressed
- [ ] CI passing

---

## References

- [Managing pull requests with Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/use-copilot-cli/manage-pull-requests)
- [Delegating tasks to Copilot cloud agent](https://docs.github.com/en/copilot/how-tos/copilot-cli/use-copilot-cli/delegate-tasks-to-cca)
- [About Copilot CLI](https://docs.github.com/en/copilot/concepts/agents/copilot-cli/about-copilot-cli)
- [GitHub Skills exercise](https://github.com/skills/create-applications-with-the-copilot-cli)

---

**Prev:** [Module 02: Core Interaction](02-core-interaction.md) | **Next:** [Module 04: Customization](04-customization.md)
