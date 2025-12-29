# Procedure: Install Serena MCP

Install and configure Serena MCP for semantic code understanding.

## What is Serena?

Serena is an MCP (Model Context Protocol) server that gives AI assistants semantic understanding of code:
- Find symbols by name across the codebase
- Understand relationships between classes/functions
- Navigate code intelligently (not just text search)
- Persistent memory for project-specific knowledge

---

## Serena MCP Tool Reference

Once installed, Serena provides these MCP tools for code analysis and memory management:

### Core Analysis Tools

| Tool Name | Purpose | Key Parameters |
|-----------|---------|----------------|
| `mcp__serena__get_symbols_overview` | Get high-level view of symbols in a file (classes, functions, etc.) | `relative_path` (required) |
| `mcp__serena__find_symbol` | Find symbols by name pattern across codebase | `name_path_pattern` (required), `relative_path` (optional), `include_body`, `depth` |
| `mcp__serena__search_for_pattern` | Regex search across files (flexible file filtering) | `substring_pattern` (required), `relative_path`, `paths_include_glob`, `paths_exclude_glob` |
| `mcp__serena__find_referencing_symbols` | Find all references to a specific symbol | `name_path` (required), `relative_path` (required) |

### Memory Tools

| Tool Name | Purpose | Key Parameters |
|-----------|---------|----------------|
| `mcp__serena__write_memory` | Store project knowledge for future sessions | `memory_file_name` (required), `content` (required) |
| `mcp__serena__read_memory` | Retrieve previously stored project knowledge | `memory_file_name` (required) |
| `mcp__serena__list_memories` | List all available memory files | (none) |
| `mcp__serena__edit_memory` | Update existing memory content | `memory_file_name`, `needle`, `repl`, `mode` |
| `mcp__serena__delete_memory` | Remove a memory file | `memory_file_name` (required) |

### Project Management Tools

| Tool Name | Purpose | Key Parameters |
|-----------|---------|----------------|
| `mcp__serena__onboarding` | Initialize Serena for a new project | (none) - interactive |
| `mcp__serena__check_onboarding_performed` | Verify if project is already configured | (none) |
| `mcp__serena__activate_project` | Switch to a different project | `project` (required) |
| `mcp__serena__list_dir` | List directory contents | `relative_path`, `recursive` |
| `mcp__serena__find_file` | Find files by name/mask | `file_mask`, `relative_path` |

### Code Modification Tools

| Tool Name | Purpose | Key Parameters |
|-----------|---------|----------------|
| `mcp__serena__replace_symbol_body` | Replace a symbol's implementation | `name_path`, `relative_path`, `body` |
| `mcp__serena__insert_after_symbol` | Insert code after a symbol | `name_path`, `relative_path`, `body` |
| `mcp__serena__insert_before_symbol` | Insert code before a symbol | `name_path`, `relative_path`, `body` |
| `mcp__serena__rename_symbol` | Rename a symbol across codebase | `name_path`, `relative_path`, `new_name` |

### Workflow Tools

| Tool Name | Purpose | Key Parameters |
|-----------|---------|----------------|
| `mcp__serena__think_about_collected_information` | Reflect on search results | (none) |
| `mcp__serena__think_about_task_adherence` | Verify still on track before edits | (none) |
| `mcp__serena__think_about_whether_you_are_done` | Check if task is complete | (none) |
| `mcp__serena__initial_instructions` | Get Serena usage manual | (none) |

---

## Prerequisites

- Python `uv` package manager (check with `uv --version`)
- Node.js installed (check with `node --version`) - for language server features

If prerequisites aren't met, log a warning and skip Serena installation.

---

## Tool-Specific Installation

Serena installation varies depending on which AI coding tool you're using. Check the `detected_tool` from the environment state.

### For Claude Code

Use the `claude mcp` CLI commands:

#### On Windows:

```bash
claude mcp add-json serena '{"type":"stdio","command":"cmd","args":["/c","uvx","--from","git+https://github.com/oraios/serena","serena","start-mcp-server","--context","ide-assistant"]}' -s user
```

#### On macOS/Linux:

```bash
claude mcp add-json serena '{"type":"stdio","command":"uvx","args":["--from","git+https://github.com/oraios/serena","serena","start-mcp-server","--context","ide-assistant"]}' -s user
```

**Verification:**
```bash
claude mcp list
```
Expected: `serena: ... - ✓ Connected`

### For Cursor

Cursor supports MCP servers through its settings:

1. Open Cursor Settings (Cmd/Ctrl + ,)
2. Navigate to "Features" → "MCP Servers"
3. Click "Add Server"
4. Configure manually with:
   - **Name**: `serena`
   - **Command**: `uvx`
   - **Args**: `--from`, `git+https://github.com/oraios/serena`, `serena`, `start-mcp-server`, `--context`, `ide-assistant`

Or edit the MCP configuration file directly (location varies by OS).

### For Windsurf

Windsurf supports MCP servers similarly to Cursor:

1. Open Windsurf Settings
2. Navigate to MCP configuration
3. Add Serena server with the same configuration as Cursor

### For Cline (VS Code Extension)

1. Open VS Code Settings (Cmd/Ctrl + ,)
2. Search for "Cline MCP"
3. Add server configuration:
   ```json
   {
     "serena": {
       "command": "uvx",
       "args": ["--from", "git+https://github.com/oraios/serena", "serena", "start-mcp-server", "--context", "ide-assistant"]
     }
   }
   ```

### For Other Tools

Provide manual configuration guidance:

1. Serena is an MCP server that runs via `uvx`
2. The command to start it: `uvx --from git+https://github.com/oraios/serena serena start-mcp-server --context ide-assistant`
3. Configure your tool's MCP settings to use this command
4. Refer to your tool's documentation for MCP server configuration

---

## Steps

### 1. Check for Existing Serena Setup [AUTO]

Check if Serena is already configured (method varies by tool):

**For Claude Code:**
```bash
claude mcp list
```

**For other tools:** Check if Serena tools are available (try calling `mcp__serena__initial_instructions`).

If Serena shows as connected/working, skip to verification step.

### 2. Check Prerequisites [AUTO]

Verify `uv` is available:
```bash
uv --version
```

- If available: continue with installation
- If not available: log warning, skip Serena, continue with other bootstrap steps

### 3. Configure Serena MCP [AUTO]

Based on the detected tool, follow the appropriate installation method from the Tool-Specific Installation section above.

### 4. Verify Installation [AUTO]

Try using a Serena tool to confirm it's working:
- Call `mcp__serena__initial_instructions` to test connectivity
- If it responds, Serena is working

### 5. Initialize Serena Project [AUTO]

Use Serena's `onboarding` tool to initialize the project automatically.

This will:
- Detect the project's language and structure
- Create `.serena/project.yml` configuration
- Set up the memories directory
- Index the codebase

### 6. Create Initial Memory [AUTO]

After Serena initializes, create a bootstrap memory using Serena's memory tools:

```markdown
# Bootstrap Information

Bootstrapped on {{date}} using AI Praxis.

## Project Analysis Results
- **Project Type**: {{project_type}}
- **Primary Language**: {{primary_language}}
- **Frameworks**: {{frameworks}}

## Guardrails Installed
- AGENTS.md with project instructions
- ai-docs/ with guides (TDD, code quality, research workflow)
- ADR structure at docs/adrs/

## Next Steps
- Review and customize AGENTS.md
- Add project-specific knowledge to memories
- Create ADRs for existing architectural decisions
```

### 7. Log Installation Status [AUTO]

Log what was accomplished:

> "Serena MCP installed and initialized:
> - `.serena/project.yml` - Project configuration
> - `.serena/memories/` - Persistent knowledge store
> - Semantic code navigation capabilities
>
> Note: Your AI coding tool may need a restart for MCP server recognition."

## Troubleshooting Reference

**Serena shows "Failed to connect":**
1. Verify `uv` is installed: `uv --version`
2. Check your tool's MCP configuration
3. Restart your AI coding tool
4. Try reinstalling with the correct configuration

**Serena initialization fails:**
- Project may have no recognizable source files
- Language may not be supported
- Try manual configuration if needed

**uvx command not found:**
- Install uv: `curl -LsSf https://astral.sh/uv/install.sh | sh` (macOS/Linux)
- Or: `pip install uv` / `pipx install uv`

## Skip Conditions

Skip this procedure if:
- User explicitly asked to skip Serena
- `uv` not available (log and continue)
- Installation fails after one attempt (log and continue)

## Error Handling

For all errors in this procedure:
1. Log the error to console
2. Record in state file
3. Continue with next bootstrap procedure

Do NOT stop the bootstrap for Serena failures - it's an optional enhancement.

## Self-Verification Checklist

Before proceeding to the next step, verify:

- [ ] `uv` availability checked
- [ ] Serena is configured in your AI tool
- [ ] If project bootstrap: `.serena/project.yml` exists
- [ ] State file updated with Serena status

Proceed to next step regardless of Serena success/failure.
