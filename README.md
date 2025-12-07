# Claude Praxis

The practice of effective AI-assisted development.

## What Is This?

**Praxis** (Greek: πρᾶξις) means "practice, as distinguished from theory" — the practical application of principles.

This repository contains a **comprehensive methodology** for Claude Code that transforms how AI assists with software development. It's not just configuration — it's a complete system of principles, workflows, and practices.

When you bootstrap a project with Claude Praxis, you get:

- **Hypothesis-Driven Development** - Test-first thinking as a core methodology
- **Structured Problem-Solving** - Research workflows and multi-approach validation
- **Semantic Code Understanding** - Serena MCP for intelligent code navigation
- **Progressive Knowledge System** - CLAUDE.md → guides → ADRs → memories
- **Project-Aware Instructions** - Customized to your actual tech stack and conventions

## How To Use

1. Open your project in Claude Code
2. Tell Claude:

```
Look at https://github.com/rosudrag/claude-praxis
and use it to bootstrap my project
```

3. Claude will:
   - Perform deep technical and cultural analysis of your project
   - Install Serena MCP for semantic code understanding
   - Generate a customized CLAUDE.md with your conventions
   - Create methodology guides (TDD, research, problem-solving)
   - Set up Architecture Decision Records

## What Gets Created

```
your-project/
├── CLAUDE.md              # AI instructions customized for your project
├── claude-docs/           # Methodology guides
│   ├── tdd-enforcement.md
│   ├── code-quality.md
│   ├── security.md
│   ├── research-workflow.md
│   ├── iterative-problem-solving.md
│   └── multi-approach-validation.md
├── docs/
│   └── adrs/              # Architecture Decision Records
│       ├── README.md
│       └── 000-template.md
├── .serena/               # Semantic code understanding
│   ├── project.yml
│   └── memories/
└── .claude/               # Claude Code configuration
    └── settings.local.json
```

## Philosophy

1. **Methodology over configuration** - A complete way of working, not just settings
2. **Hypothesis-driven** - Tests are hypotheses; state expectations, then prove them
3. **Research before assuming** - Investigate systematically, don't guess
4. **Progressive disclosure** - Layer information: quick reference → detailed guides → historical context
5. **Project-aware** - Adapt to the actual codebase, not generic best practices

## Customization

After bootstrapping:

- Edit `CLAUDE.md` to add project-specific instructions
- Add custom guides to `claude-docs/`
- Create ADRs for architectural decisions
- Add Serena memories for persistent knowledge

## Requirements

- Claude Code CLI or VS Code extension
- Node.js (for Serena MCP installation)
- Git

## Contributing

See [CLAUDE.md](CLAUDE.md) for development instructions.

## License

MIT
