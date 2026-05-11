# Module 04: Customization, Instructions, Agents, and Skills

**Duration:** 90 minutes | **Prev:** [Module 03: GitHub Integration](03-github-integration.md) | **Next:** [Module 05: MCP and LSP](05-mcp-and-lsp.md)

## Learning Objectives

- Set up repository-wide and global custom instructions
- Create path-specific instruction files with `applyTo` frontmatter
- Invoke built-in agents and create custom agents with YAML frontmatter
- Understand what skills are and how they differ from agents

---

## 4.1 Custom Instructions Hierarchy (30 min)

Custom instructions tell Copilot about your codebase, standards, and workflow. They persist across every session in that repo.

**Discovery order (repository overrides global):**

| File location | Scope | Notes |
|---|---|---|
| `~/.copilot/copilot-instructions.md` | All sessions globally | Your personal defaults |
| `.github/copilot-instructions.md` | Repository-wide | Team standards, commit to repo |
| `.github/instructions/**/*.instructions.md` | Path-specific | Use `applyTo:` frontmatter |
| `AGENTS.md` (repo root or cwd) | Repository | Also read by Copilot cloud agent |
| `CLAUDE.md` / `GEMINI.md` / `CODEX.md` | Repository | Model-specific overrides |

**What to put in repository instructions:**

```markdown
## Build Commands
- `npm run build`: builds to dist/
- `npm run test`: runs Jest test suite
- `npm run lint:fix`: runs ESLint with auto-fix
- `npm run typecheck`: runs tsc --noEmit

## Code Style
- TypeScript strict mode, no implicit any
- Functional components only, no class components
- JSDoc on all exported functions
- Conventional commits format for commit messages

## Workflow
- Run `npm run lint:fix && npm test` after every change
- Create feature branches from `main`, never commit directly to main
- Always link PRs to the relevant issue using `Closes #N`

## Architecture
- API routes in src/api/, one file per resource
- Business logic in src/services/, stateless functions only
- Database queries in src/db/, use the QueryBuilder wrapper, never raw SQL
```

**Path-specific instructions** (`.github/instructions/api.instructions.md`):

```markdown
---
applyTo: "src/api/**/*.ts"
excludeAgent: "code-review"
---
All API handlers must:
- Include try/catch error boundaries
- Return errors in the format: { error: string, code: string }
- Use pino for structured logging, never console.log
- Validate all inputs with zod schemas before processing
```

Another example (`.github/instructions/tests.instructions.md`):

```markdown
---
applyTo: "**/*.test.ts"
---
All tests must follow the AAA pattern: Arrange, Act, Assert.
Use `describe` blocks to group related tests.
Mock all external dependencies (databases, APIs), no real network calls in tests.
Use `expect.objectContaining()` rather than exact object matching where appropriate.
```

**Viewing active instructions:**

```bash
/instructions    # list loaded instruction files and toggle them on/off
/env             # show all loaded context: instructions, MCP, skills, agents
```

**Disable instructions for a session:**

```bash
copilot --no-custom-instructions
```

---

## 4.2 Built-in Custom Agents (30 min)

Agents are specialist personas with specific tools and context windows. Invoke with `/agent` or by describing the task.

| Agent | Purpose | Tools available | When to invoke |
|---|---|---|---|
| `Explore` | Codebase research, doesn't touch main context | grep, glob, view | Understanding unfamiliar code without side effects |
| `Task` | Runs commands, brief on success, verbose on failure | shell, view | Running tests, builds, lints, CI-like validation |
| `General purpose` | Complex multi-step work in its own context window | All | Long-running side work you don't want in main context |
| `Code review` | High-signal review, only surfaces genuine bugs | read, shell(git) | Pre-PR security and logic review |
| `Research` | Deep codebase + web research with citations | grep, web_fetch, github | Architecture decisions, library comparisons |
| `Rubber duck` | Constructive critic, used internally by Copilot | N/A | Auto-invoked by Copilot when it needs a second opinion |

Invocation patterns:

```bash
/agent                                    # browse and select from list
Use the code-review agent to review my changes
Use the explore agent to find all places where we authenticate users
copilot --agent=code-review -p "/review"  # programmatic
```

---

## 4.3 Creating Custom Agents (20 min)

Create `.md` files with YAML frontmatter in these locations:

- `~/.copilot/agents/` — user-level, available in all repos
- `.github/agents/` — repository-level
- `/agents/` in `.github-private` repo — org/enterprise-level

Example: security auditor agent (`.github/agents/security-auditor.md`):

```markdown
---
name: Security Auditor
description: Reviews code changes for OWASP vulnerabilities and compliance issues
model: claude-opus-4.5
tools: [read, shell(git diff)]
---
You are a security-focused code reviewer. When reviewing code, check for:

## OWASP Top 10
- SQL injection: look for string concatenation in queries
- XSS: look for unescaped output in HTML/templates
- CSRF: check form endpoints for token validation
- Hardcoded secrets: scan for API keys, passwords, connection strings

## Output Format
For each finding:
- File and line number
- Risk level: Critical / High / Medium / Low
- Exact code snippet
- Recommended fix

Only report genuine findings. No "looks good" commentary for clean code sections.
```

Example: TypeScript specialist agent (`.github/agents/ts-specialist.md`):

```markdown
---
name: TypeScript Specialist
description: Implements TypeScript features following the project's strict typing conventions
model: claude-sonnet-4.5
tools: [read, write, shell(npm run:*)]
---
This project uses TypeScript strict mode. When writing code:
- Use explicit return types on all functions
- Prefer type unions over `any`
- Use `satisfies` operator for type-safe object literals
- Prefer discriminated unions for state modeling
- Run `npm run typecheck` after every change

Do not use `as` type assertions unless absolutely necessary, and always add a comment explaining why.
```

---

## 4.4 Skills (10 min)

Skills provide just-in-time task-specific instructions. Unlike static instructions, which are always loaded, skills activate on demand and can include executable scripts.

Discovery locations:

- `~/.copilot/skills/` — user-level
- `.github/skills/` — repository-level

Managing skills:

```bash
/skills list          # all available skills
/skills info NAME     # details on a specific skill
/skills reload        # reload after adding new skill files
```

Skills can be bundled into plugins, covered in [Module 06: Automation](06-automation.md).

---

## Lab 4: Instructions and Custom Agent (20 min)

**Goal:** Set up project instructions and a custom agent.

**Steps:**

1. Create `.github/copilot-instructions.md` for the calculator project with:
   - Build and test commands
   - Code style guide (use JSDoc, no `var`, prefer `const`)
   - Workflow conventions
2. Create a path-specific instruction file `.github/instructions/tests.instructions.md` applying the AAA pattern to all test files
3. Run `/instructions` to verify both files are loaded
4. Create a custom agent file `.github/agents/calculator-expert.md` that:
   - Specializes in mathematical correctness
   - Requires checking edge cases: negative numbers, very large values, NaN
   - Uses the `code-review` approach but focused on math accuracy
5. Invoke the agent: "Use the calculator-expert agent to review src/calculator.js"
6. Compare the review output vs. what you'd get without the agent context
7. Run `/env` to see the full loaded context

**Validation:**

- [ ] Custom instructions visible in `/instructions`
- [ ] Custom agent invoked successfully
- [ ] `/env` shows all loaded context

---

## References

- [Add custom instructions](https://docs.github.com/en/copilot/how-tos/copilot-cli/add-custom-instructions)
- [About custom agents](https://docs.github.com/en/copilot/concepts/agents/coding-agent/about-custom-agents)
- [About agent skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills)

---

**Prev:** [Module 03: GitHub Integration](03-github-integration.md) | **Next:** [Module 05: MCP and LSP](05-mcp-and-lsp.md)
