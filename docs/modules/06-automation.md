# Module 06: Automation and Programmatic Use

**Duration:** 120 minutes | **Prev:** [Module 05: MCP and LSP](05-mcp-and-lsp.md) | **Next:** [Module 07: Hooks and Governance](07-hooks-and-governance.md)

## Learning Objectives

- Run Copilot CLI in headless/programmatic mode for scripting
- Apply tool permission flags to create safe, bounded automation
- Integrate Copilot CLI into GitHub Actions pipelines
- Understand the plugin system for packaging and distributing extensions

---

## 6.1 Programmatic Mode (30 min)

The `-p` flag switches Copilot CLI from interactive to headless mode: one prompt, one response, exits.

### Core Flags

```bash
# Single prompt, exits after
copilot -p "PROMPT"

# Silent output, suppresses usage info, ideal for scripts
copilot -sp "PROMPT"

# Interactive session with auto-executed opening prompt
copilot -i "PROMPT"

# Change working directory without cd
copilot -C /path/to/project -p "Fix all TypeScript errors"

# Enable autopilot continuation, default: 5 continues
copilot --autopilot -p "Refactor the authentication module"
copilot --max-autopilot-continues=15 -p "Add tests to every untested file"

# No user prompts, fully autonomous, use with care
copilot --no-ask-user -p "Run tests and fix failures"
```

### Output Formats

```bash
# Human-readable text, default
copilot -p "Summarize recent commits" --output-format=text

# Structured JSONL, parse in scripts
copilot -p "List all TODO comments in src/" --output-format=json
```

### Context Attachment in Programmatic Mode (v1.0.41+)

```bash
# Include an image or file as context
copilot -p "Describe this architecture diagram" --attachment=./diagram.png

# Include multiple attachments
copilot -p "Review these two files together" --attachment=file1.ts --attachment=file2.ts
```

### Session Sharing from Programmatic Mode

```bash
# Save session to file after completion
copilot -p "Audit dependencies for vulnerabilities" \
  --allow-tool='shell(npm:*)' \
  --share='./audit-report.md'

# Save as GitHub Gist
copilot -p "Summarize project architecture" --share-gist
```

---

## 6.2 Tool Permission Flags (30 min)

Every tool execution requires approval in interactive mode. In programmatic mode, use flags to pre-approve or pre-deny categories.

### Allow Flags

| Flag | Allows |
|---|---|
| `--allow-all-tools` | All tools without prompting |
| `--allow-tool='shell(git:*)'` | All git subcommands |
| `--allow-tool='shell(npm run:*)'` | All npm run scripts |
| `--allow-tool='shell(npm run test:*)'` | Only test-related scripts |
| `--allow-tool='write'` | All file writes without prompting |
| `--allow-tool='My-MCP-Server'` | All tools from a named MCP server |
| `--allow-all-paths` | Disable file path verification |
| `--allow-all-urls` | Allow all URLs without confirmation |

### Deny Flags (override allow)

| Flag | Blocks |
|---|---|
| `--deny-tool='shell(rm)'` | The `rm` command |
| `--deny-tool='shell(git push)'` | Git push |
| `--deny-tool='shell(curl)'` | All curl calls |
| `--deny-tool='shell(sudo)'` | All sudo calls |

### Safe Patterns for Common Scenarios

**Test runner (allow tests, deny destructive operations):**

```bash
copilot -p "Run tests and fix any failures" \
  --allow-tool='shell(npm run test:*)' \
  --allow-tool='shell(git:*)' \
  --deny-tool='shell(git push)' \
  --deny-tool='shell(rm)' \
  --no-ask-user
```

**Code linter (allow linting and file writes, nothing else):**

```bash
copilot -p "Run the linter and fix all auto-fixable issues" \
  --allow-tool='shell(npm run lint:fix)' \
  --allow-tool='write' \
  --deny-tool='shell(git)' \
  --no-ask-user
```

**Full automation in a sandboxed container:**

```bash
copilot --allow-all-tools \
        --deny-tool='shell(rm)' \
        --deny-tool='shell(git push)' \
        --deny-tool='shell(curl)' \
        --no-ask-user \
        -p "Implement the feature and run all tests"
```

**Read-only exploration:**

```bash
copilot -p "Analyze this codebase and identify all security anti-patterns" \
  --allow-tool='shell(git log)' \
  --deny-tool='write' \
  --deny-tool='shell(npm)' \
  --no-ask-user
```

---

## 6.3 GitHub Actions Integration (40 min)

Authentication in CI: use a fine-grained PAT with "Copilot Requests" permission stored as a secret.

```bash
export COPILOT_GITHUB_TOKEN=${{ secrets.COPILOT_PAT }}
```

### Example 1: AI-Powered Test Fixer

```yaml
# .github/workflows/ai-test-fixer.yml
name: AI Test Fixer
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  fix-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
      - name: Install Copilot CLI
        run: npm install -g @github/copilot
      - name: Fix failing tests
        env:
          COPILOT_GITHUB_TOKEN: ${{ secrets.COPILOT_PAT }}
        run: |
          copilot -p "Run the test suite. Fix any failing tests by editing source or test files as needed. Commit fixes." \
            --allow-tool='shell(npm:*)' \
            --allow-tool='shell(git:*)' \
            --deny-tool='shell(git push)' \
            --no-ask-user
```

### Example 2: PR Code Review Comment

```yaml
# .github/workflows/ai-review.yml
name: AI Code Review
on:
  pull_request:
    types: [opened]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: actions/setup-node@v4
        with: { node-version: '22' }
      - run: npm install -g @github/copilot
      - name: Generate review
        env:
          COPILOT_GITHUB_TOKEN: ${{ secrets.COPILOT_PAT }}
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          REVIEW=$(copilot -sp "Review the diff of this PR compared to main. Focus only on bugs, security issues, and logic errors. Output in plain text." \
            --allow-tool='shell(git:*)' \
            --no-ask-user)
          gh pr comment ${{ github.event.pull_request.number }} --body "$REVIEW"
```

### Example 3: Weekly Dependency Audit

```yaml
# .github/workflows/ai-dep-audit.yml
name: Dependency Audit
on:
  schedule:
    - cron: '0 9 * * 1'  # Monday mornings

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '22' }
      - run: npm install -g @github/copilot
      - name: Audit and create issue
        env:
          COPILOT_GITHUB_TOKEN: ${{ secrets.COPILOT_PAT }}
        run: |
          copilot -p "Run npm audit. Summarize any high or critical vulnerabilities. Create a GitHub issue titled 'Weekly Dependency Audit' with the findings." \
            --allow-tool='shell(npm audit)' \
            --enable-all-github-mcp-tools \
            --no-ask-user
```

### CI-Specific Environment Variables

```bash
# Opt-in for CI: enable repo hooks and workspace MCP in prompt mode (v1.0.41+)
GITHUB_COPILOT_PROMPT_MODE_REPO_HOOKS=true
GITHUB_COPILOT_PROMPT_MODE_WORKSPACE_MCP=true

# Disable telemetry for air-gapped or private environments
COPILOT_OFFLINE=true

# Override the CLI config directory
COPILOT_HOME=/path/to/config
```

---

## 6.4 Plugin System (20 min)

Plugins bundle multiple capabilities into a single distributable package:

- Custom agents
- Skills
- Hooks
- MCP server configurations

### Managing Plugins

```bash
# Command-line management
copilot plugin list             # list installed plugins
copilot plugin update           # update all plugins to latest versions

# Add a plugin from a git repository
copilot plugin install git@github.com:owner/my-plugin-repo
```

In interactive sessions:

```bash
/plugin list                    # list installed plugins
/plugin install git@github.com:owner/repo
/plugin update                  # update all
/plugin uninstall plugin-name
/plugin marketplace             # browse available plugins
```

### Plugin Structure

A plugin repository contains:

```
plugin.json            # manifest: name, version, description
agents/                # custom agent .md files
skills/                # skill definition files
hooks/                 # hook configuration and scripts
mcp-config.json        # MCP server definitions to add
```

---

## Lab 6: CI/CD Integration (30 min)

**Goal:** Create a GitHub Actions workflow that uses Copilot CLI for code quality automation.

**Steps:**

1. Create `.github/workflows/ai-review.yml` in the calculator project
2. The workflow should trigger on PRs and:
   - Run the test suite
   - Use Copilot to review changes compared to main
   - Post the review as a PR comment
3. Use `--deny-tool='shell(git push)'` and `--deny-tool='shell(rm)'` as safety guards
4. Test by opening a small PR with a deliberate bug in the code
5. Verify the AI comment appears on the PR
6. Bonus: add a second workflow that runs on a schedule to audit dependencies

**Validation:**

- [ ] PR comment appears with AI-generated review
- [ ] No push or delete operations attempted

---

## References

- [Run Copilot CLI programmatically](https://docs.github.com/en/copilot/how-tos/copilot-cli/automate-copilot-cli/run-cli-programmatically)
- [CLI command reference](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference)

---

**Prev:** [Module 05: MCP and LSP](05-mcp-and-lsp.md) | **Next:** [Module 07: Hooks and Governance](07-hooks-and-governance.md)
