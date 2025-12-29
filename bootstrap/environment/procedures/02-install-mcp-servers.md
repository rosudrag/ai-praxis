# Procedure: Install MCP Servers

Install essential MCP servers for enhanced AI-assisted development.

## What Are MCP Servers?

MCP (Model Context Protocol) servers extend AI assistant capabilities by providing:
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

## Tool-Specific Installation

MCP server installation varies depending on which AI coding tool you're using. Check the `detected_tool` from the environment state.

### For Claude Code

Use the `claude mcp` CLI commands:

#### Add Context7:

```bash
claude mcp add-json context7 '{"type":"stdio","command":"npx","args":["-y","@upstash/context7-mcp@latest"]}' -s user
```

#### Add Sequential Thinking:

```bash
claude mcp add-json sequential-thinking '{"type":"stdio","command":"npx","args":["-y","@modelcontextprotocol/server-sequential-thinking"]}' -s user
```

**Verification:**
```bash
claude mcp list
```
Expected output shows servers as "Connected":
```
context7: npx -y @upstash/context7-mcp@latest - ✓ Connected
sequential-thinking: npx -y @modelcontextprotocol/server-sequential-thinking - ✓ Connected
```

### For Cursor

Cursor supports MCP servers through its settings:

1. Open Cursor Settings (Cmd/Ctrl + ,)
2. Navigate to "Features" → "MCP Servers"
3. Add servers:

**Context7:**
- **Name**: `context7`
- **Command**: `npx`
- **Args**: `-y`, `@upstash/context7-mcp@latest`

**Sequential Thinking:**
- **Name**: `sequential-thinking`
- **Command**: `npx`
- **Args**: `-y`, `@modelcontextprotocol/server-sequential-thinking`

### For Windsurf

Windsurf supports MCP servers similarly to Cursor:

1. Open Windsurf Settings
2. Navigate to MCP configuration
3. Add the same server configurations as Cursor

### For Cline (VS Code Extension)

1. Open VS Code Settings (Cmd/Ctrl + ,)
2. Search for "Cline MCP"
3. Add server configurations:
   ```json
   {
     "context7": {
       "command": "npx",
       "args": ["-y", "@upstash/context7-mcp@latest"]
     },
     "sequential-thinking": {
       "command": "npx",
       "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
     }
   }
   ```

### For Other Tools

Provide manual configuration guidance:

**Context7:**
- Command: `npx -y @upstash/context7-mcp@latest`
- Type: stdio

**Sequential Thinking:**
- Command: `npx -y @modelcontextprotocol/server-sequential-thinking`
- Type: stdio

Refer to your tool's documentation for MCP server configuration.

---

## Steps

### 1. Check for Existing MCP Configuration [AUTO]

Check if servers are already configured (method varies by tool):

**For Claude Code:**
```bash
claude mcp list
```

**For other tools:** Check if the tools are available by testing their functionality.

If context7 and/or sequential-thinking are already configured and connected, skip to verification step.

### 2. Check Node.js Prerequisites [AUTO]

Verify Node.js 18+ and npm are available:
- If available: continue with installation
- If not available: log warning, skip MCP servers, continue with other bootstrap steps

### 3. Configure MCP Servers [AUTO]

Based on the detected tool, follow the appropriate installation method from the Tool-Specific Installation section above.

### 4. Verify Installation [AUTO]

Confirm servers are working:
- For Claude Code: `claude mcp list` shows Connected status
- For other tools: Test by using the tools if available

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
> Note: Your AI coding tool may need a restart for MCP server recognition."

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
- Both servers already configured

## Error Handling

For all errors in this procedure:
1. Log the error to console
2. Record in state file
3. Continue with next bootstrap procedure

Do NOT stop the bootstrap for MCP server failures - they are optional enhancements.

## Self-Verification Checklist

Before proceeding to the next step, verify:

- [ ] Node.js availability checked
- [ ] Context7 server configured
- [ ] Sequential Thinking server configured
- [ ] State file updated with MCP servers status

Proceed to next step regardless of MCP server success/failure.

## Troubleshooting Reference

**MCP servers show "Failed to connect":**
1. Check your tool's MCP configuration
2. Verify the command and args are correct
3. Restart your AI coding tool

**npx command fails:**
- Check internet connectivity
- npm cache may need clearing: `npm cache clean --force`
- Try installing packages directly

**Context7 not working:**
- Ensure you include "use context7" in prompts
- Library may not be in Context7's database yet
- Try specifying the Context7 library ID directly

**Sequential Thinking not activating:**
- Tool is used automatically for complex reasoning
- Can be explicitly invoked for structured problem-solving
- Check configuration status
