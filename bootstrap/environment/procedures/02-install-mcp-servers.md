# Procedure: Install MCP Servers

Install essential MCP servers for enhanced AI-assisted development.

## What Are MCP Servers?

MCP (Model Context Protocol) servers extend Claude's capabilities by providing:
- Access to up-to-date documentation (Context7)
- Structured reasoning for complex problems (Sequential Thinking)
- Integration with external tools and services

## MCP Servers Installed

### Context7

**Purpose**: Provides up-to-date, version-specific library documentation directly in prompts.

**Benefits**:
- Eliminates outdated or hallucinated API information
- Injects real documentation for libraries you're using
- Supports 1000+ popular libraries and frameworks

**Usage**: Add "use context7" to prompts, e.g., "How do I use React Server Components? use context7"

### Sequential Thinking

**Purpose**: Enables structured, step-by-step reasoning for complex problems.

**Benefits**:
- Breaks down complex problems into manageable steps
- Supports thought revision as understanding evolves
- Enables branching for alternative reasoning paths
- Maintains context over multiple reasoning steps

**Best for**:
- Planning and design tasks
- Debugging complex issues
- Analysis requiring course correction
- Problems where full scope isn't initially clear

---

## Prerequisites

- Node.js 18+ installed (check with `node --version`)
- npm available (check with `npm --version`)

If prerequisites aren't met, log a warning and skip MCP server installation.

---

## CRITICAL: MCP Configuration Location

**DO NOT manually edit configuration files.** Use the `claude mcp` CLI commands instead.

### Where MCP Servers Are Configured

MCP servers are stored in `~/.claude.json` under the `mcpServers` key (for user-scoped servers) or under `projects.<project-path>.mcpServers` (for project-scoped servers).

| Scope | Location | Availability |
|-------|----------|--------------|
| **user** | `~/.claude.json` → `mcpServers` | All projects |
| **local** | `~/.claude.json` → `projects.<path>.mcpServers` | Single project only |
| **project** | `.mcp.json` in project root | Shared via git |

### WRONG Locations (Do NOT Use)

These files are NOT read for MCP configuration:
- `~/.claude/settings.local.json` - Does NOT configure MCP servers
- `~/.claude/.mcp/user.json` - Legacy/internal, not reliable
- `~/.claude/settings.json` - For other settings, not MCP

---

## Steps

### 1. Check for Existing MCP Configuration [AUTO]

Use the CLI to check existing servers:

```bash
claude mcp list
```

If context7 and/or sequential-thinking are already configured and connected, skip to verification step.

### 2. Check Node.js Prerequisites [AUTO]

Verify Node.js 18+ and npm are available:
- If available: continue with installation
- If not available: log warning, skip MCP servers, continue with other bootstrap steps

### 3. Configure MCP Servers [AUTO]

**ALWAYS use `claude mcp add-json` for reliable configuration.**

The `claude mcp add` command with `--` separator can mangle arguments (especially on Windows where `/c` flags get misinterpreted).

#### Add Context7 (user scope - available in all projects):

```bash
claude mcp add-json context7 '{"type":"stdio","command":"npx","args":["-y","@upstash/context7-mcp@latest"]}' -s user
```

#### Add Sequential Thinking (user scope):

```bash
claude mcp add-json sequential-thinking '{"type":"stdio","command":"npx","args":["-y","@modelcontextprotocol/server-sequential-thinking"]}' -s user
```

### 4. Verify Installation [AUTO]

```bash
claude mcp list
```

Expected output shows all servers as "Connected":
```
context7: npx -y @upstash/context7-mcp@latest - ✓ Connected
sequential-thinking: npx -y @modelcontextprotocol/server-sequential-thinking - ✓ Connected
```

### 5. Log Installation Status [AUTO]

Log what was accomplished:

> "MCP servers configured:
>
> **Context7** - Up-to-date library documentation
> - Usage: Add 'use context7' to prompts for current docs
> - Example: 'How do I use Next.js App Router? use context7'
>
> **Sequential Thinking** - Structured reasoning
> - Automatically used for complex multi-step problems
> - Helps with planning, debugging, and analysis tasks
>
> Note: Claude Code restart may be needed for MCP server recognition."

## MCP Server Reference

Once installed, these MCP servers provide the following capabilities:

### Context7 Tools

| Tool Name | Purpose | Key Parameters |
|-----------|---------|----------------|
| `resolve-library-id` | Resolves library name to Context7 ID | `libraryName` (required) |
| `get-library-docs` | Fetches documentation for a library | `context7CompatibleLibraryID` (required), `topic` (optional), `tokens` (optional, default 5000) |

### Sequential Thinking Tools

| Tool Name | Purpose | Key Parameters |
|-----------|---------|----------------|
| `sequentialthinking` | Record and track reasoning steps | `thought`, `thoughtNumber`, `totalThoughts`, `nextThoughtNeeded`, `isRevision`, `revisesThought`, `branchFromThought`, `branchId` |

---

## Skip Conditions

Skip this procedure if:
- User explicitly asked to skip MCP servers
- Node.js/npm not available (log and continue)
- Both servers already configured (check with `claude mcp list`)

## Error Handling

For all errors in this procedure:
1. Log the error to console
2. Record in state file
3. Continue with next bootstrap procedure

Do NOT stop the bootstrap for MCP server failures - they are optional enhancements.

## Self-Verification Checklist

Before proceeding to the next step, verify:

- [ ] Node.js availability checked
- [ ] `claude mcp list` shows servers as "Connected"
- [ ] Context7 server configured
- [ ] Sequential Thinking server configured
- [ ] State file updated with MCP servers status

Proceed to next step regardless of MCP server success/failure.

## Troubleshooting Reference

**MCP servers show "Failed to connect":**
1. Check the server configuration: `claude mcp get <server-name>`
2. Verify the command and args are correct
3. Remove and re-add: `claude mcp remove <name> -s user` then `claude mcp add-json ...`
4. Restart Claude Code

**Wrong configuration location:**
- MCP is configured in `~/.claude.json`, NOT in `~/.claude/settings.local.json`
- Use `claude mcp` commands, never manually edit files

**Windows-specific: Arguments mangled (e.g., `/c` becomes `C:/`):**
- Do NOT use `claude mcp add ... -- cmd /c ...`
- ALWAYS use `claude mcp add-json` with explicit JSON config
- For Windows commands needing `cmd /c`, use:
  ```bash
  claude mcp add-json myserver '{"type":"stdio","command":"cmd","args":["/c","your","command","here"]}' -s user
  ```

**Context7 not working:**
- Ensure you include "use context7" in prompts
- Library may not be in Context7's database yet
- Try specifying the Context7 library ID directly

**Sequential Thinking not activating:**
- Tool is used automatically for complex reasoning
- Can be explicitly invoked for structured problem-solving
- Check `claude mcp list` for connection status

**npx command fails:**
- Check internet connectivity
- npm cache may need clearing: `npm cache clean --force`
- Try installing packages directly: `npm install -g @upstash/context7-mcp`

## CLI Reference

| Command | Purpose |
|---------|---------|
| `claude mcp list` | List all configured servers and their status |
| `claude mcp get <name>` | Show detailed config for a server |
| `claude mcp add-json <name> '<json>' -s <scope>` | Add server with explicit JSON (recommended) |
| `claude mcp remove <name> -s <scope>` | Remove a server |
| `claude mcp reset-project-choices` | Reset approval choices for project-scoped servers |
