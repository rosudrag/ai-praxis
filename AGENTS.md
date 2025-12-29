# AI Praxis - Development Instructions

This file contains instructions for developing and improving THIS repository itself.

> **If you're here to bootstrap another project**, see [bootstrap/AGENTS.md](bootstrap/AGENTS.md) instead.

## Project Purpose

This repository is a **comprehensive methodology system** for AI coding assistants. When users point their AI tool at this repo, it reads the instructions in `/bootstrap` and uses them to set up the praxis system in the user's own project.

**Supported Tools**: Claude Code, Cursor, Windsurf, Cline, and other MCP-compatible AI assistants.

## Repository Structure

```
ai-praxis/
├── AGENTS.md                    # You are here (meta instructions)
├── CLAUDE.md                    # Forwarding file for Claude Code
├── README.md                    # Human documentation
└── bootstrap/
    ├── AGENTS.md                # Entry point router (mode selection)
    ├── .state/                  # Environment setup state
    │   ├── .gitkeep
    │   └── environment-state.template.json
    ├── environment/             # Mode 1: Environment setup (global tools)
    │   ├── AGENTS.md           # Environment orchestrator
    │   └── procedures/
    │       ├── 01-install-serena.md
    │       ├── 02-install-mcp-servers.md
    │       └── 03-install-agents.md
    ├── project/                 # Mode 2: Project bootstrap (project files)
    │   ├── AGENTS.md           # Project orchestrator
    │   ├── procedures/
    │   │   ├── 00-analyze-project.md
    │   │   ├── 01-generate-agents-md.md
    │   │   ├── 02-generate-docs.md
    │   │   └── 03-setup-adrs.md
    │   └── verification.md      # Post-bootstrap verification
    ├── templates/               # Shared templates for both modes
    │   ├── AGENTS.md.template
    │   ├── adr-template.md
    │   └── guides/              # Guide templates (TDD, code quality, etc.)
    └── reference/               # Shared reference docs
        ├── guardrail-principles.md
        └── serena-best-practices.md
```

## Development Guidelines

### When Editing Procedures
- Each procedure should be self-contained and executable
- Use clear step numbering (1, 2, 3...)
- Include decision points where the AI should ask the user
- Mark actions as `[AUTO]`, `[CONFIRM]`, or `[INTERACTIVE]`
- Include tool-specific instructions for different AI tools (Claude Code, Cursor, etc.)

### When Editing Templates
- Use `{{placeholder}}` syntax for dynamic content
- Use `{{#if condition}}...{{/if}}` for conditional sections
- Mark user-preservable sections with `<!-- USER_SECTION_START -->` / `<!-- USER_SECTION_END -->`

### When Adding New Features
1. Determine which mode it affects:
   - Environment setup → `/bootstrap/environment/procedures/`
   - Project bootstrap → `/bootstrap/project/procedures/`
   - Both → update relevant orchestrators
2. Add any new templates to `/bootstrap/templates/`
3. Update the appropriate orchestrator (environment/AGENTS.md or project/AGENTS.md)
4. Update `/bootstrap/AGENTS.md` if mode routing logic changes
5. Update this file if the repo structure changes

### Tool-Agnostic Considerations
- MCP installation varies by tool - include instructions for each supported tool
- Use generic terms like "AI coding assistant" instead of "Claude Code" where appropriate
- The bootstrap generates both `AGENTS.md` (primary) and `CLAUDE.md` (forwarding file for Claude Code)
- Docs directory is `ai-docs/` not `claude-docs/`
- Bootstrap state is `.ai-bootstrap/` not `.claude-bootstrap/`

## Testing Changes

To test bootstrap changes:
1. Create a test repository
2. Point your AI coding tool at this repo and ask it to bootstrap the test repo
3. Verify the generated files match expectations
4. Check idempotency by running bootstrap again
5. Test with different AI tools if possible (Claude Code, Cursor, etc.)

## Commit Guidelines

- Use 50/72 rule for commit messages
- First line: imperative summary (max 50 chars)
- Second line: blank
- Third line onwards: details if needed (wrap at 72 chars)
