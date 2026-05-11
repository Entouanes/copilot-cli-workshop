# Lab Project Guide

[Course Home](../index.md) | [Quick Reference Cheatsheet](cheatsheet.md)

The course uses a single Node.js project that grows across all 8 modules. Starting from a simple calculator, it evolves into a full-featured app with tests, CI/CD, database integration, and a governance policy.

---

## 1. Repository Setup

```bash
# Fork or create a new repo named: copilot-cli-course-project
# Clone it locally
git clone https://github.com/YOUR_USERNAME/copilot-cli-course-project
cd copilot-cli-course-project
```

---

## 2. Initial Project Structure

```
copilot-cli-course-project/
├── .devcontainer/
│   └── devcontainer.json
├── .github/
│   ├── ISSUE_TEMPLATE/
│   │   └── feature_request.md
│   ├── workflows/
│   │   └── (empty initially)
│   ├── hooks/
│   │   └── (created in Module 7)
│   ├── agents/
│   │   └── (created in Module 4)
│   └── instructions/
│       └── (created in Module 4)
├── src/
│   ├── calculator.js          # created in Module 2
│   └── tests/
│       └── calculator.test.js # created in Module 2
├── data/
│   └── .gitkeep
├── images/
│   └── js-calculator.png
├── package.json
└── README.md
```

---

## 3. Initial Files to Create

### `package.json`

```json
{
  "name": "copilot-cli-course-project",
  "version": "1.0.0",
  "description": "Node.js calculator app built with GitHub Copilot CLI",
  "main": "src/calculator.js",
  "scripts": {
    "test": "jest",
    "test:coverage": "jest --coverage",
    "lint": "eslint src/",
    "lint:fix": "eslint src/ --fix"
  },
  "devDependencies": {
    "jest": "^29.0.0",
    "eslint": "^8.0.0"
  }
}
```

### `.github/ISSUE_TEMPLATE/feature_request.md`

```markdown
---
name: Feature Request
about: Request a new feature for the calculator
title: ''
labels: enhancement
---

## Feature Description
<!-- What feature would you like? -->

## Use Case
<!-- Why is this needed? -->

## Proposed Solution
<!-- How might it be implemented? -->

## Additional Context
<!-- Anything else? Edge cases, examples -->
```

### `.devcontainer/devcontainer.json`

```json
{
  "name": "Copilot CLI Course",
  "image": "mcr.microsoft.com/devcontainers/javascript-node:1-22-bookworm",
  "postCreateCommand": "npm install",
  "features": {
    "ghcr.io/devcontainers/features/github-cli:1": {}
  },
  "customizations": {
    "vscode": {
      "extensions": ["GitHub.copilot"]
    }
  }
}
```

### `README.md`

```markdown
# Copilot CLI Course Project

A Node.js calculator app built step-by-step using GitHub Copilot CLI.

## Prerequisites

- Node.js 22+
- GitHub Copilot subscription (Pro, Pro+, Business, or Enterprise)
- GitHub Copilot CLI: `npm install -g @github/copilot`

## Installation

npm install

## Usage

node src/calculator.js

## Testing

npm test
```

---

## 4. What Gets Built Per Module

| Module | What's added | Key files |
|---|---|---|
| 1 (Foundations) | Setup only | `package.json`, `README.md` |
| 2 (Core Interaction) | Basic calculator | `src/calculator.js`, `src/tests/calculator.test.js` |
| 3 (GitHub Integration) | History feature, PRs | `src/calculator.js` (updated) |
| 4 (Customization) | Instructions, custom agent | `.github/copilot-instructions.md`, `.github/agents/` |
| 5 (MCP and LSP) | SQLite history DB | `data/history.db`, updated calculator |
| 6 (Automation) | CI workflows | `.github/workflows/ai-review.yml` |
| 7 (Hooks and Governance) | Hook pipeline | `.github/hooks/policies.json`, `scripts/` |
| 8 (Capstone) | Scientific mode | `src/calculator.js` (scientific functions) |

---

## 5. Codespace Setup

If you prefer using GitHub Codespaces instead of a local environment:

1. Push the initial project to GitHub
2. Go to the repository on GitHub.com
3. Click **Code** → **Codespaces** → **Create codespace on main**
4. In the terminal: `npm install -g @github/copilot`
5. Authenticate: `copilot` then `/login`

Codespaces come with Node.js 22 pre-installed. The `devcontainer.json` handles all setup automatically.

---

[Course Home](../index.md) | [Quick Reference Cheatsheet](cheatsheet.md)
