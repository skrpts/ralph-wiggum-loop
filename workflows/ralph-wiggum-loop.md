---
type: workflow
id: ralph-wiggum-loop
title: Ralph Wiggum Loop
description: "Autonomous development loop — pick a task, implement, test, commit, repeat until all tasks pass"
tags: [Production, Customer-Facing, Developer, Loop]
connections:
  - target: task-selection
    type: uses
  - target: implementation
    type: uses
  - target: test-verification
    type: uses
  - target: completion-report
    type: uses
  - target: llm-service
    type: runs_on
  - target: progress-tracker-template
    type: references
  - target: loop-runner-script
    type: references
  - target: claude-code-config
    type: references
metadata:
  estimated_duration: "10-60 minutes"
  trigger: manual
  loop_modes: ["until_pass"]
execution:
  - skill: "task-selection"
    step_type: "synthesis"
  - skill: "implementation"
    step_type: "generation"
  - skill: "test-verification"
    step_type: "validation"
  - skill: "completion-report"
    step_type: "synthesis"
---

## Overview

This workflow implements the **Ralph Wiggum** autonomous development pattern — an `until_pass` loop where each iteration selects an incomplete task, implements it, runs tests, and commits the result. The verifier checks whether all tasks are complete and tests pass.

Named after the technique popularised by Geoffrey Huntley for running AI coding agents in autonomous loops, the pattern works by giving each iteration a fresh context window with a progress file as the handoff mechanism between iterations.

## Pipeline Stages

### Stage 1: Development Loop (until_pass, max 10 iterations)

Each iteration runs three steps:

**Step 1 — Task Selection**

Reads the progress tracker file and selects the next incomplete task. Considers task dependencies and priority. On the first iteration, validates the task list and confirms the starting point. On subsequent iterations, reads the updated progress to find what remains.

**Step 2 — Implementation**

Implements the selected task: writes code, updates configuration, modifies tests. Follows the project conventions from the Claude Code config. After implementation, runs the project's test suite and build commands. Commits the changes with a descriptive message.

**Step 3 — Test Verification (verifier)**

Evaluates whether the current task was completed successfully and whether all tasks are now done. Produces a structured verdict:

- `pass` — all tasks complete and tests passing. Loop ends.
- `continue` — current task done but more tasks remain, or current task needs another attempt. Includes instructions for the next iteration.
- `fail` — unrecoverable problem detected (e.g. fundamentally broken architecture, missing dependencies that can't be installed).

### Stage 2: Completion Report (post-loop)

After the loop ends, a final step produces a summary of what was accomplished: which tasks were completed, how many iterations it took, any tasks that remain incomplete, and a log of the key decisions made during implementation.

## Inputs

| Name | Required | Description | Example |
|------|----------|-------------|---------|
| `{{input.project_path}}` | Yes | Path to the project root | `/home/user/my-app` |
| `{{input.task_list}}` | Yes | Tasks to complete (markdown checklist or description) | "Add user authentication with JWT tokens, write tests, update API docs" |
| `{{input.test_command}}` | Yes | Command to run the test suite | `npm test` or `pytest` or `cargo test` |
| `{{input.build_command}}` | No | Command to verify the build (defaults to test command) | `npm run build` |
| `{{input.conventions}}` | No | Project-specific coding conventions | "Use TypeScript strict mode, prefer functional style, test with vitest" |

## Outputs

| Name | Description |
|------|-------------|
| Progress tracker | Updated task list with completion status per task |
| Git commits | One commit per completed task with descriptive messages |
| Completion report | Summary of work done, iterations taken, and remaining items |

## Setup

Before running this workflow:

1. Ensure the project has a working test suite. The loop relies on tests to verify implementation correctness.
2. Prepare a clear task list. Vague tasks ("improve performance") produce vague implementations. Specific tasks ("add Redis caching to the /api/users endpoint") get done.
3. Set a sensible iteration limit. Start with 5 for small task lists, up to 10 for larger ones. Each iteration consumes a full AI context window.
4. Review the Claude Code config asset for recommended CLAUDE.md conventions.

## Provider Notes

- Each iteration runs three AI calls (select + implement + verify). A 10-iteration loop = 30 AI calls plus the completion report.
- Fresh context is enabled — each iteration starts clean, reading project state from disk rather than carrying forward a growing conversation.
- The implementation step uses shell access for running tests and git commands. The `shell:execute` permission is required.
- Cost scales linearly with iterations. A 10-iteration run on a medium codebase typically costs $10-30 in API usage depending on the model.

## Safety

- **Max iterations are mandatory.** The loop stops after the configured limit even if tasks remain.
- **Each iteration commits its work.** If the loop is interrupted, completed tasks are preserved in git history.
- **The verifier can fail the loop.** If a task is fundamentally impossible or the codebase is in an unrecoverable state, the verifier returns `fail` to stop wasting iterations.
- **Progress is tracked on disk.** The progress file acts as a checkpoint — you can review what happened, adjust the task list, and restart the loop.

## Example Input

```
project_path: "/home/user/my-express-app"
task_list: |
  - [ ] Add input validation middleware using zod
  - [ ] Add rate limiting to all API endpoints
  - [ ] Write integration tests for the auth flow
  - [ ] Update OpenAPI spec to match current endpoints
test_command: "npm test"
build_command: "npm run build"
conventions: "TypeScript strict, ESM imports, vitest for tests, no default exports"
```
