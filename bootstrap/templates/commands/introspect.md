# /introspect - Audit Past Conversations for Improvements

Analyze recent Claude Code conversation transcripts to identify friction points, missing docs, skill gaps, and behavioral corrections.

## Instructions

### 1. Gather Transcripts

List recent conversation transcripts sorted by size (larger = more work = more signal):

```bash
ls -lS "{{transcripts_path}}" --time=ctime | grep ".jsonl" | head -15
```

Pick the 5-8 largest/most recent transcripts for analysis.

### 2. Extract User Messages

For each transcript, extract user messages to understand what was asked, corrected, and repeated:

```python
import json
with open('<transcript>.jsonl', 'r', encoding='utf-8', errors='ignore') as f:
    for line in f:
        d = json.loads(line)
        if d.get('type') == 'user':
            content = d.get('message', {}).get('content', '')
            # Extract text from string or list content
```

Look for patterns:
- **Corrections**: "no", "wrong", "not that", "I said", "stop"
- **Repeated asks**: Same question asked 2+ times
- **Interruptions**: "[Request interrupted by user]"
- **Frustration signals**: profanity, ALL CAPS, "!!!", short terse corrections
- **Manual workflows**: Multi-step operations the user guided step-by-step

### 3. Spawn Analysis Agents

For large transcripts (>500KB), spawn background agents to analyze each one in parallel. Each agent should extract:

1. **Tasks/topics discussed**
2. **Friction points** — things the user had to repeat, correct, or re-explain
3. **Manual workflows** that could be automated with a skill command
4. **Missing documentation** that would have prevented confusion
5. **User corrections/feedback** — exact quotes when possible

### 4. Cross-Reference Existing Setup

Check what already exists before recommending duplicates:
- Existing skills: `{{commands_path}}/*.md`
- Existing memories: `{{memory_path}}/*.md`
- AGENTS.md / CLAUDE.md rules
- Serena memories (if installed): `.serena/memories/*.md`

### 5. Categorize Findings

Group into:

**New Skill Commands** — repeated manual workflows (3+ occurrences across conversations)
**New Memories** — behavioral corrections, project rules, or reference info learned the hard way
**Doc Updates** — existing skills/docs that need fixes or additions
**AGENTS.md Rules** — patterns important enough to be in the root instructions

### 6. Implement

For each finding:
- **Skills**: Create `{{commands_path}}/<name>.md` with full instructions
- **Feedback memories**: Create `memory/feedback_<name>.md` with rule + why + how to apply
- **Project memories**: Create `memory/project_<name>.md` with fact + why + how to apply
- **Doc fixes**: Edit existing files directly
- **Update MEMORY.md index** with links to all new memory files

### 7. Report

Present a summary table of everything created/updated, grouped by type:

```
| Type     | Name               | Action  | Description                        |
|----------|--------------------|---------|-------------------------------------|
| Command  | /deploy            | Created | Automated deployment workflow       |
| Memory   | feedback_testing   | Created | Always run integration tests first  |
| Doc      | AGENTS.md          | Updated | Added missing review checklist      |
```

## When to Run

- After a busy week of development (5+ conversations)
- When you notice the same friction point appearing across sessions
- When the user explicitly asks to improve the setup
- Periodically as a hygiene check

## Constraints

- Focus on actionable improvements, not theoretical ones
- Prioritize by frequency x impact: a small friction that happens every session beats a big one that happened once
- Don't create memories for things derivable from code (patterns, architecture) — only for behavioral rules and non-obvious project knowledge
- Keep MEMORY.md under 200 lines — it's loaded into every conversation
- Don't create duplicate memories — check existing ones first
