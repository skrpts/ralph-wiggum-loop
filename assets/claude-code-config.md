---
type: asset
id: claude-code-config
title: Claude Code Configuration
description: "Recommended CLAUDE.md conventions and settings for autonomous loop operation"
tags: [Production, Developer, Configuration]
connections: []
metadata:
  asset_type: "reference"
  format: "markdown"
---

# Claude Code Configuration for Ralph Wiggum Loops

When running the Ralph Wiggum pattern, your project's `CLAUDE.md` file needs specific conventions to support autonomous iteration. This reference covers what to include and why.

## Recommended CLAUDE.md Structure

```markdown
# Project Name

## What This Is
[One paragraph — what the project does, what stack it uses]

## Commands
- **Test:** `npm test`
- **Build:** `npm run build`
- **Lint:** `npm run lint`
- **Single test:** `npm test -- --grep "test name"`

## Conventions
- [Language/framework conventions]
- [File naming patterns]
- [Import style]
- [Error handling approach]

## Architecture
- [Directory structure overview]
- [Key abstractions]
- [Data flow]

## Autonomous Operation Rules
- Read PROGRESS.md at the start of every session
- Select ONE incomplete task and implement it fully
- Run tests after every change — do not commit failing code
- Update PROGRESS.md after completing or failing a task
- Commit each completed task separately with a descriptive message
- Do not modify code unrelated to the current task
- If blocked, mark the task with [!] and explain why in the notes
- Exit cleanly after completing or failing one task
```

## Key Sections Explained

### Commands

List every command the agent needs. The AI should not have to guess how to run tests or build the project. Include commands for running individual tests — the agent often needs to run a specific test file rather than the full suite.

### Conventions

Be explicit about coding style. The agent follows what it's told, not what it infers. If you want TypeScript strict mode, say so. If you want functional style over classes, say so. The more specific, the more consistent the output.

### Autonomous Operation Rules

This section is critical for loop operation. Without it, the agent may:

- Try to complete all tasks in one session (context bloat)
- Skip tests to move faster
- Make unrelated "improvements" to code it reads
- Not update the progress file (breaking the handoff)

The rules above prevent these failure modes. Adapt them to your project, but keep the core pattern: read state → do one thing → verify → update state → exit.

## Permission Settings

For autonomous operation, Claude Code needs permission to:

- Read and write files in the project directory
- Execute shell commands (tests, build, git)
- Access git (commit, log, diff)

Consider using an allowlist for shell commands:

```
Allowed tools: Read, Write, Edit, Bash(git *), Bash(npm *), Bash(cargo *), Bash(pytest *)
```

This prevents the agent from running arbitrary shell commands while still allowing the tools it needs.

## Cost Awareness

Each loop iteration is a full Claude Code session. Costs depend on:

- **Codebase size** — larger codebases mean more tokens for context
- **Task complexity** — complex tasks may require more tool calls within a single iteration
- **Model choice** — Opus costs more per token than Sonnet

A rough guide for a medium codebase (10-50 files):

| Iterations | Estimated cost |
|-----------|---------------|
| 5 | $5-15 |
| 10 | $10-30 |
| 20 | $20-60 |

Monitor your API usage dashboard during early runs. Adjust `MAX_ITERATIONS` based on actual costs.
