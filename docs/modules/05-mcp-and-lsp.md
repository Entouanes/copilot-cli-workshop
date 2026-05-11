# Module 05: MCP Servers and Language Intelligence

**Duration:** 90 minutes | **Prev:** [Module 04: Customization](04-customization.md) | **Next:** [Module 06: Automation](06-automation.md)

## Learning Objectives

- Explain MCP and why it matters for extending Copilot's reach beyond code
- Add, configure, and troubleshoot external MCP servers
- Set up LSP servers for code intelligence inside the terminal session
- Use session-scoped MCP servers for one-off integrations

---

## 5.1 MCP Explained (20 min)

**Model Context Protocol (MCP)** is a standard interface for connecting AI models to external services and data sources. Often described as "USB-C for AI": one protocol, many compatible services.

Copilot CLI ships with a built-in GitHub MCP server, covered in [Module 03](03-github-integration.md). Beyond that, you can connect:

- Databases: PostgreSQL, SQLite, MySQL
- File systems outside the current working directory
- Slack, Jira, Linear, Notion
- Custom internal services via the MCP SDK
- Any service that exposes an MCP-compatible server

How it works:

1. You configure an MCP server in `~/.copilot/mcp-config.json`
2. Copilot CLI starts the server process at session launch
3. The AI can discover and call the server's tools like any other built-in tool
4. Tool calls require the same approval flow as built-in tools

---

## 5.2 Managing MCP Servers (40 min)

### Interactive Management

```bash
/mcp add          # fill form: name, command, args; Ctrl+S to save
/mcp show         # view status, connected tools, and stderr diagnostics
/mcp edit         # modify an existing server config
/mcp delete       # remove a server
/mcp reload       # restart all servers, pick up config changes
/mcp auth         # authenticate to a server requiring OAuth
/mcp disable      # disable without deleting
/mcp enable       # re-enable a disabled server
```

Configuration stored at `~/.copilot/mcp-config.json`.

### Configuration Examples

#### PostgreSQL Database

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres", "postgresql://localhost/mydb"],
      "env": {}
    }
  }
}
```

Usage: "Query the users table and show me the 5 most recently created accounts"

#### Local Filesystem (read access to projects outside cwd)

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/me/projects"]
    }
  }
}
```

Usage: "Read the API contract from /Users/me/projects/shared-contracts/user-api.yaml"

#### Slack Integration

```json
{
  "mcpServers": {
    "slack": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "xoxb-...",
        "SLACK_TEAM_ID": "T..."
      }
    }
  }
}
```

Usage: "Look at the #backend-team channel and summarize the discussion about the auth refactor"

#### SQLite (local database)

```json
{
  "mcpServers": {
    "sqlite": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sqlite", "--db-path", "./data/dev.db"]
    }
  }
}
```

#### Custom Internal Service

```json
{
  "mcpServers": {
    "internal-docs": {
      "command": "python",
      "args": ["-m", "my_mcp_server"],
      "cwd": "/path/to/server",
      "env": {
        "API_KEY": "...",
        "BASE_URL": "https://docs.internal.company.com"
      }
    }
  }
}
```

### Using MCP Tools with OAuth (v1.0.40+)

For servers requiring OAuth:

```bash
/mcp auth server-name    # opens browser for OAuth flow
```

`client_credentials` grant type is supported for headless or CI use (no browser required).

### Session-Scoped MCP Server

Add an MCP server for one session without modifying persistent config:

```bash
copilot --additional-mcp-config='{"mcpServers":{"local-db":{"command":"./scripts/db-mcp.sh"}}}'
```

Useful for project-specific one-off integrations that should not be global.

### Troubleshooting

```bash
/mcp show          # check connection status and stderr output
```

Common issues:

| Issue | Fix |
|---|---|
| "Server not found" | Verify the command is in PATH (`which npx` or `which python`) |
| "Connection refused" | Server process may be crashing; check stderr in `/mcp show` |
| Tool calls failing | Check that the server version supports the tool schema being called |

---

## 5.3 LSP Integration: Code Intelligence (30 min)

**Language Server Protocol (LSP)** gives Copilot CLI the same code understanding as your IDE: go-to-definition, hover documentation, diagnostics, type information.

- Without LSP: Copilot reads files as plain text
- With LSP: Copilot understands semantic structure — it knows what type `userId` is, what methods are available on an object, and which files export which symbols

### Configuration

- User-level (all repos): `~/.copilot/lsp-config.json`
- Repo-level (committed): `.github/lsp.json`

```json
{
  "lspServers": {
    "typescript": {
      "command": "typescript-language-server",
      "args": ["--stdio"],
      "fileExtensions": {
        ".ts": "typescript",
        ".tsx": "typescript"
      }
    },
    "python": {
      "command": "pylsp",
      "args": [],
      "fileExtensions": {
        ".py": "python"
      }
    },
    "rust": {
      "command": "rust-analyzer",
      "args": ["--stdio"],
      "fileExtensions": {
        ".rs": "rust"
      }
    }
  }
}
```

Prerequisites: the language server binary must be installed separately:

```bash
# TypeScript
npm install -g typescript-language-server typescript

# Python
pip install python-lsp-server

# Rust
rustup component add rust-analyzer
```

### Managing LSP in Session

```bash
/lsp               # show active servers and loaded files
/lsp test          # verify connectivity to all configured servers
/lsp reload        # restart servers after config changes
/lsp show SERVER   # detailed status for a specific server
```

### What LSP Enables

- More accurate refactoring suggestions (Copilot knows all call sites)
- Better error diagnosis (understands type mismatches, not just text patterns)
- Smarter import resolution suggestions
- More accurate "find all usages" exploration
- Go-to-definition style responses when asked about symbols

---

## Lab 5: MCP and Code Intelligence (20 min)

**Goal:** Connect an external data source and set up language intelligence.

**Part A: MCP**

1. Install the SQLite MCP server: `npm install -g @modelcontextprotocol/server-sqlite`
2. Create a simple SQLite database for the calculator project:
   ```bash
   sqlite3 ./data/history.db "CREATE TABLE calculations (id INTEGER PRIMARY KEY, expression TEXT, result REAL, timestamp DATETIME DEFAULT CURRENT_TIMESTAMP);"
   ```
3. Add it to `~/.copilot/mcp-config.json`
4. Start a session, run `/mcp show` to verify the connection
5. Ask Copilot to insert a few test calculations into the database via natural language
6. Ask Copilot to query and summarize the calculation history

**Part B: LSP**

1. Install TypeScript LSP: `npm install -g typescript-language-server typescript`
2. Add it to `~/.copilot/lsp-config.json` for `.js` files, or rename `calculator.js` to `.ts`
3. Run `/lsp test` to confirm connectivity
4. Ask Copilot to refactor a function and observe how it handles cross-file impact
5. Compare the quality of suggestions before and after LSP is active

**Validation:**

- [ ] MCP server connected and tools accessible (`/mcp show`)
- [ ] LSP server verified with `/lsp test`

---

## References

- [CLI command reference](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference)
- [Copilot CLI repository](https://github.com/github/copilot-cli)
- [Model Context Protocol](https://modelcontextprotocol.io)

---

**Prev:** [Module 04: Customization](04-customization.md) | **Next:** [Module 06: Automation](06-automation.md)
