# Procedure: Install Serena MCP

Install and configure Serena MCP for semantic code understanding.

## What is Serena?

Serena is an MCP (Model Context Protocol) server that gives Claude semantic understanding of code:
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

## CRITICAL: MCP Configuration

**DO NOT manually edit configuration files.** Use the `claude mcp` CLI commands.

Serena is a Python-based MCP server installed via `uvx` from GitHub, NOT via npm.

---

## Steps

### 1. Check for Existing Serena Setup [AUTO]

Check if Serena is already configured:

```bash
claude mcp list
```

If serena shows as "Connected", skip to verification step.

### 2. Check Prerequisites [AUTO]

Verify `uv` is available:
```bash
uv --version
```

- If available: continue with installation
- If not available: log warning, skip Serena, continue with other bootstrap steps

### 3. Configure Serena MCP [AUTO]

**ALWAYS use `claude mcp add-json` for reliable configuration.**

#### On Windows:

```bash
claude mcp add-json serena '{"type":"stdio","command":"cmd","args":["/c","uvx","--from","git+https://github.com/oraios/serena","serena","start-mcp-server","--context","ide-assistant"]}' -s user
```

#### On macOS/Linux:

```bash
claude mcp add-json serena '{"type":"stdio","command":"uvx","args":["--from","git+https://github.com/oraios/serena","serena","start-mcp-server","--context","ide-assistant"]}' -s user
```

### 4. Verify Installation [AUTO]

```bash
claude mcp list
```

Expected output:
```
serena: ... - ✓ Connected
```

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

Bootstrapped on {{date}} using Claude Praxis.

## Project Analysis Results
- **Project Type**: {{project_type}}
- **Primary Language**: {{primary_language}}
- **Frameworks**: {{frameworks}}

## Guardrails Installed
- CLAUDE.md with project instructions
- claude-docs/ with guides (TDD, code quality, research workflow)
- ADR structure at docs/adrs/

## Next Steps
- Review and customize CLAUDE.md
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
> Note: Claude Code restart may be needed for MCP server recognition."

## Troubleshooting Reference

**Serena shows "Failed to connect":**
1. Check configuration: `claude mcp get serena`
2. Verify `uv` is installed: `uv --version`
3. Remove and re-add with correct config (see Step 3)
4. Restart Claude Code

**Wrong configuration location:**
- MCP is configured in `~/.claude.json`, NOT in `~/.claude/settings.local.json`
- NEVER manually edit files - use `claude mcp` commands

**Windows-specific: Arguments mangled (e.g., `/c` becomes `C:/`):**
- Do NOT use `claude mcp add ... -- cmd /c ...`
- ALWAYS use `claude mcp add-json` with explicit JSON config

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
- [ ] `claude mcp list` shows serena as "Connected"
- [ ] If project bootstrap: `.serena/project.yml` exists
- [ ] State file updated with Serena status

Proceed to next step regardless of Serena success/failure.

## CLI Reference

| Command | Purpose |
|---------|---------|
| `claude mcp list` | List all configured servers and their status |
| `claude mcp get serena` | Show detailed config for Serena |
| `claude mcp add-json serena '<json>' -s user` | Add Serena with explicit JSON |
| `claude mcp remove serena -s user` | Remove Serena configuration |
