# Procedure: Generate CLAUDE.md

Create a customized CLAUDE.md file for the user's project.

## Purpose

CLAUDE.md is the primary instruction file that Claude Code reads when working on a project. It should contain:
- Project overview and context
- Key architectural decisions
- Navigation guides
- Coding standards
- Links to detailed documentation

## Steps

### 1. Check for Existing CLAUDE.md [AUTO]

If `CLAUDE.md` already exists in the project root:
- **Default action**: Merge - Add guardrails sections while preserving user content
- Create backup as `CLAUDE.md.backup` before any modifications

### 2. Load Template [AUTO]

Read the template from [templates/CLAUDE.md.template](../templates/CLAUDE.md.template).

If template not found, generate a minimal CLAUDE.md from scratch.

### 3. Customize Template [AUTO]

Process the template by replacing placeholders with values from the analysis file (`.claude-bootstrap/analysis.json`).

#### Basic Variable Substitution

Replace these placeholders with actual values:

| Placeholder | Replace With |
|-------------|--------------|
| `{{project_name}}` | Project directory name, or `name` from package.json/*.csproj if available |
| `{{primary_language}}` | Value from analysis (e.g., "TypeScript", "C#", "Python") |
| `{{frameworks}}` | Comma-separated list from analysis (e.g., "React, Next.js") |
| `{{project_type}}` | Value from analysis (e.g., "Application", "Library", "Monorepo") |
| `{{date}}` | Current date in YYYY-MM-DD format |

#### Discovery Variable Substitution

The template uses discovery data from analysis to auto-populate paths and commands.

**Note:** Discovery values use a nested structure with `.value`, `.source`, and `.confidence` fields:
```json
{
  "discovery": {
    "key_paths": {
      "entry_point": { "value": "src/index.ts", "confidence": "high" }
    }
  }
}
```

**Key Paths:**

| Placeholder | Replace With | Fallback |
|-------------|--------------|----------|
| `{{discovery.key_paths.entry_point.value}}` | Discovered entry point path | Show "Not detected" |
| `{{discovery.key_paths.business_logic.value}}` | Discovered business logic path | Show "Not detected" |
| `{{discovery.key_paths.tests.value}}` | Discovered tests path | Show "Not detected" |
| `{{discovery.key_paths.config.value}}` | Discovered config path | Show "Not detected" |

**Commands:**

| Placeholder | Replace With | Fallback |
|-------------|--------------|----------|
| `{{discovery.commands.build.value}}` | Discovered build command | Show placeholder block |
| `{{discovery.commands.test.value}}` | Discovered test command | Show placeholder block |
| `{{discovery.commands.lint.value}}` | Discovered lint command | Omit section entirely |
| `{{discovery.commands.dev.value}}` | Discovered dev server command | Omit section entirely |

**Project Purpose:**

| Placeholder | Replace With | Fallback |
|-------------|--------------|----------|
| `{{discovery.project_purpose.value}}` | Extracted description | Show placeholder block |

**Conventions:**

| Placeholder | Replace With |
|-------------|--------------|
| `{{discovery.conventions.linter}}` | Detected linter (e.g., "eslint") |
| `{{discovery.conventions.formatter}}` | Detected formatter (e.g., "prettier") |
| `{{discovery.conventions.naming_style}}` | Naming convention description |
| `{{discovery.conventions.import_order}}` | Import organization pattern |
| `{{discovery.conventions.quote_style}}` | Quote preference (single/double) |
| `{{discovery.conventions.semicolons}}` | Semicolon preference (true/false) |
| `{{discovery.conventions.indent}}` | Indentation style (e.g., "2 spaces") |

**Patterns:**

| Placeholder | Replace With |
|-------------|--------------|
| `{{discovery.patterns.error_handling}}` | Error handling approach |
| `{{discovery.patterns.logging}}` | Logging pattern |
| `{{discovery.patterns.testing}}` | Testing style/pattern |
| `{{discovery.patterns.documentation}}` | Documentation style |

#### Required Variables with Fallbacks

The following variables MUST have values. Use these fallbacks if detection fails:

| Variable | Fallback Strategy |
|----------|-------------------|
| `{{project_name}}` | Use directory name |
| `{{primary_language}}` | Use "Unknown" (user can edit later) |
| `{{project_type}}` | Default to "Application" |
| `{{frameworks}}` | Default to "None detected" |
| `{{date}}` | Current date in YYYY-MM-DD format |

#### Conditional Section Rules

For sections wrapped in `{{#if condition}}...{{else}}...{{/if}}`:

- **Include the if section** if the condition/value exists and is truthy
- **Include the else section** if the condition/value is missing or falsy
- **Remove the entire block** (including the if/else tags) after processing

#### Unless Blocks

`{{#unless condition}}...{{/unless}}` is the inverse of `{{#if}}`:
- **Include the block** if the condition is missing, null, undefined, empty string, or false
- **Exclude the block** if the condition exists and is truthy

#### Template Processing Algorithm

When processing templates, follow this exact algorithm:

**Step 1: Variable Substitution**

Replace `{{variable.path}}` with values using dot notation:
- `{{project_name}}` -> lookup `analysis.project_name`
- `{{discovery.key_paths.entry_point.value}}` -> lookup `analysis.discovery.key_paths.entry_point.value`

**Step 2: Evaluate Truthiness**

A value is considered **truthy** if:
- It exists AND
- It is not `null` or `undefined` AND
- It is not an empty string `""` AND
- It is not an empty array `[]` AND
- It is not `false`

A value is considered **falsy** if any of the above conditions fail.

**Step 3: Process Conditional Blocks**

Process conditionals from innermost to outermost (nested blocks first):

1. Find `{{#if condition}}`, `{{#unless condition}}` blocks
2. Evaluate the condition's truthiness
3. For `{{#if}}`:
   - If truthy: keep content between `{{#if}}` and `{{else}}` (or `{{/if}}` if no else)
   - If falsy: keep content between `{{else}}` and `{{/if}}` (or nothing if no else)
4. For `{{#unless}}`:
   - If falsy: keep the content
   - If truthy: remove the content
5. Remove all `{{#if}}`, `{{else}}`, `{{/if}}`, `{{#unless}}`, `{{/unless}}` tags

**Step 4: Cleanup**

- Remove any remaining template tags
- Remove any blank lines created by removed conditional blocks (but preserve intentional spacing)
- Ensure no `{{` or `}}` patterns remain in output (except in code blocks demonstrating template syntax)

**Step 5: Validation**

- Scan output for unresolved `{{placeholder}}` patterns
- If found outside of `{{else}}` placeholder blocks, log as warning but continue

### 4. Log Discovery Summary [AUTO]

Log what was discovered (informational, non-blocking):

> "Generating CLAUDE.md with discovered values:
>
> **Project:** {{project_name}} ({{primary_language}})
> **Commands detected:** build, test, lint
> **Paths detected:** entry_point, tests
> **Placeholders remaining:** [list any that couldn't be detected]"

### 5. Write CLAUDE.md [AUTO]

Write the generated content to `CLAUDE.md` in the project root.

If replacing an existing file, first copy it to `CLAUDE.md.backup`.

### 6. Log Completion [AUTO]

Log what was created:

> "Created CLAUDE.md with:
> - Project Overview with detected language/frameworks
> - Build/Test commands from project config
> - Navigation guide with discovered paths
> - Auto-Discovered Configuration table
>
> Sections marked with `<!-- ... -->` comments may need manual customization."

## Template Location

The template is at: `bootstrap/templates/CLAUDE.md.template`

## Merge Strategy

When merging with existing CLAUDE.md:

1. **Preserve**: Everything between `<!-- USER_SECTION_START: name -->` and `<!-- USER_SECTION_END -->`
2. **Preserve**: Content in placeholder sections if user has edited them:
   - `<!-- PROJECT_DESCRIPTION_START -->` / `<!-- PROJECT_DESCRIPTION_END -->`
   - `<!-- BUILD_COMMANDS_START -->` / `<!-- BUILD_COMMANDS_END -->`
   - `<!-- TEST_COMMANDS_START -->` / `<!-- TEST_COMMANDS_END -->`
   - `<!-- KEY_PATHS_START -->` / `<!-- KEY_PATHS_END -->`
   - `<!-- CODING_STANDARDS_START -->` / `<!-- CODING_STANDARDS_END -->`
   - `<!-- PROJECT_NOTES_START -->` / `<!-- PROJECT_NOTES_END -->`
3. **Update**: Standard sections (Documentation Index, Core Principles)
4. **Add**: New sections that don't exist
5. **Never delete**: User's custom content

### Detecting User Edits in Placeholder Sections

To determine if a user has edited a placeholder section:
1. Read content between the START/END markers
2. Compare against the default placeholder text from the template
3. If different from default, preserve user's content
4. If same as default (or empty), can be regenerated with new discovery data

## Output

- Creates or updates `CLAUDE.md` in project root
- If replaced, creates `CLAUDE.md.backup`
- Updates `.claude-bootstrap/manifest.json` with generation timestamp

## Error Handling

### Template Not Found

If template file is missing:
1. Log warning
2. Generate minimal CLAUDE.md from scratch:
   ```markdown
   # {{project_name}}

   **Primary Language**: {{primary_language}}
   **Frameworks**: {{frameworks}}

   ## Documentation
   - [TDD Guide](claude-docs/tdd-enforcement.md)
   - [Code Quality](claude-docs/code-quality.md)
   - [Research Workflow](claude-docs/research-workflow.md)
   ```

### Cannot Write File

If file write fails:
1. Log error with details
2. If permission error, try creating backup first
3. Record failure in manifest
4. Continue with next procedure

### Analysis Data Missing

If `.claude-bootstrap/analysis.json` is missing or invalid:
1. Use fallback values for all placeholders
2. Log warning about missing analysis
3. Continue with generation using defaults

## Self-Verification Checklist

Before proceeding to the next step, verify:

- [ ] Template loaded (or fallback content generated)
- [ ] Placeholders resolved (with values or fallbacks)
- [ ] `CLAUDE.md` written to project root
- [ ] If existing file was found, backup was created
- [ ] Manifest updated with claude_md generation status

Proceed to next step regardless of which fallbacks were used.
