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
| `iterative-problem-solving.md` | Systematic debugging methodology | [templates/guides/iterative-problem-solving.md](../templates/guides/iterative-problem-solving.md) |
| `multi-approach-validation.md` | Evaluating multiple solutions | [templates/guides/multi-approach-validation.md](../templates/guides/multi-approach-validation.md) |

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

### 7. Generate Iterative Problem Solving Guide [AUTO if missing]

Copy from template:

```markdown
# Iterative Problem Solving Guide

A structured approach to solving complex problems through hypothesis formation, testing, and refinement.
```

See [templates/guides/iterative-problem-solving.md](../templates/guides/iterative-problem-solving.md).

### 8. Generate Multi-Approach Validation Guide [AUTO if missing]

Copy from template:

```markdown
# Multi-Approach Validation Guide

When the solution isn't obvious, explore multiple approaches before committing to one.
```

See [templates/guides/multi-approach-validation.md](../templates/guides/multi-approach-validation.md).

### 9. Create Index [AUTO]

Create `claude-docs/README.md`:

```markdown
# Claude Documentation

Supporting guides for AI-assisted development.

## Available Guides

### Core Practices
- [TDD Enforcement](tdd-enforcement.md) - Hypothesis-driven test-first development
- [Code Quality](code-quality.md) - Coding standards and principles
- [Security](security.md) - Security practices and guidelines

### Problem Solving
- [Iterative Problem Solving](iterative-problem-solving.md) - Systematic debugging methodology
- [Multi-Approach Validation](multi-approach-validation.md) - Evaluating multiple solutions
- [Research Workflow](research-workflow.md) - How to investigate unknowns

## Adding Custom Guides

Add any project-specific guides to this directory and reference them in the root CLAUDE.md.
```

If `claude-docs/README.md` already exists, skip it (preserve user's index).

### 10. Log Completion [AUTO]

Log what was created:

> "Created claude-docs/ with guides:
> - `tdd-enforcement.md` - {{created/skipped}}
> - `code-quality.md` - {{created/skipped}}
> - `security.md` - {{created/skipped}}
> - `research-workflow.md` - {{created/skipped}}
> - `iterative-problem-solving.md` - {{created/skipped}}
> - `multi-approach-validation.md` - {{created/skipped}}
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
- [ ] Core guides present: `tdd-enforcement.md`, `code-quality.md`, `security.md`
- [ ] Problem-solving guides present: `iterative-problem-solving.md`, `multi-approach-validation.md`, `research-workflow.md`
- [ ] `claude-docs/README.md` exists
- [ ] Manifest updated with docs generation status

Proceed to next step regardless of which files were skipped.
