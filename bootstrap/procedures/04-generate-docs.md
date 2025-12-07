# Procedure: Generate Documentation

Create the claude-docs/ directory with supporting guides.

## Purpose

The `claude-docs/` directory contains detailed guides that CLAUDE.md references. These provide in-depth instructions for specific practices without cluttering the main file.

## Standard Documents

Create these files in `claude-docs/`:

| File | Purpose | Template |
|------|---------|----------|
| `tdd-enforcement.md` | TDD workflow and requirements | [templates/guides/tdd-enforcement.md](../templates/guides/tdd-enforcement.md) |
| `code-quality.md` | Code standards and principles | [templates/guides/code-quality.md](../templates/guides/code-quality.md) |
| `security.md` | Security practices and guidelines | [templates/guides/security.md](../templates/guides/security.md) |
| `research-workflow.md` | When/how to investigate unknowns | [templates/guides/research-workflow.md](../templates/guides/research-workflow.md) |

## Steps

### 1. Create Directory [AUTO]

```bash
mkdir -p claude-docs
```

If directory creation fails (e.g., `claude-docs` exists as a file), log error and skip this procedure.

### 2. Check for Existing Docs [AUTO]

For each standard document, check if it already exists:
- If exists: **Skip** (don't overwrite user customizations)
- If missing: Create from template

### 3. Generate TDD Guide [AUTO if missing]

Copy from template, customize for detected language:

```markdown
# TDD Enforcement Guide

[Language-specific examples based on {{primary_language}}]
```

See [templates/guides/tdd-enforcement.md](../templates/guides/tdd-enforcement.md).

### 4. Generate Code Quality Guide [AUTO if missing]

Copy from template, customize for detected stack:

```markdown
# Code Quality Principles

[Stack-specific patterns based on {{frameworks}}]
```

See [templates/guides/code-quality.md](../templates/guides/code-quality.md).

### 5. Generate Security Guide [AUTO if missing]

Copy from template, customize for detected stack:

```markdown
# Security Practices

[Stack-specific security considerations based on {{frameworks}}]
```

See [templates/guides/security.md](../templates/guides/security.md).

### 6. Generate Research Workflow [AUTO if missing]

Copy from template:

```markdown
# Research Workflow

When uncertain about implementation details...
```

See [templates/guides/research-workflow.md](../templates/guides/research-workflow.md).

### 7. Create Index [AUTO]

Create `claude-docs/README.md`:

```markdown
# Claude Documentation

Supporting guides for AI-assisted development.

## Available Guides

- [TDD Enforcement](tdd-enforcement.md) - Test-driven development workflow
- [Code Quality](code-quality.md) - Coding standards and principles
- [Security](security.md) - Security practices and guidelines
- [Research Workflow](research-workflow.md) - How to investigate unknowns

## Adding Custom Guides

Add any project-specific guides to this directory and reference them in the root CLAUDE.md.
```

If `claude-docs/README.md` already exists, skip it (preserve user's index).

### 8. Log Completion [AUTO]

Log what was created:

> "Created claude-docs/ with guides:
> - `tdd-enforcement.md` - {{created/skipped}}
> - `code-quality.md` - {{created/skipped}}
> - `security.md` - {{created/skipped}}
> - `research-workflow.md` - {{created/skipped}}
> - `README.md` - {{created/skipped}}
>
> Customize these guides for your project's specific needs."

## Error Handling

### Cannot Create Directory

If `mkdir -p claude-docs` fails:
1. Log error with details
2. Record in manifest: `"docs": { "skipped": true, "reason": "directory_creation_failed" }`
3. Continue with next procedure

### Template Files Not Found

If templates are missing:
1. Log which templates are missing
2. Create minimal versions with basic content
3. Continue with available templates

### Individual File Write Fails

If one file fails while others succeed:
1. Log which file failed and why
2. Record partial progress in manifest
3. Continue with remaining files

### Disk Full

If disk full error occurs:
1. Log error
2. Record which files succeeded in manifest
3. Continue with next procedure

## Output

- Creates `claude-docs/` directory
- Creates standard guide files (if not existing)
- Creates README.md index (if not existing)

## Self-Verification Checklist

Before proceeding to the next step, verify:

- [ ] `claude-docs/` directory exists
- [ ] For each guide: either created new or existing file preserved
- [ ] `claude-docs/README.md` exists
- [ ] Manifest updated with docs generation status

Proceed to next step regardless of which files were skipped.
