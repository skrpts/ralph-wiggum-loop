---
type: skill
id: implementation
title: Implementation
description: "Implements the selected task — writes code, runs tests, and commits the changes"
tags: [Production, Developer, Code]
connections:
  - target: llm-service
    type: runs_on
---

## Capability

Takes the selected task and implements it end-to-end: writes or modifies source code, updates tests, runs the test suite, and commits the result. Follows the project's coding conventions and existing patterns.

The implementation step is the workhorse of each iteration. It operates on the actual codebase — reading files, writing code, executing shell commands — not on abstract descriptions.

## When to Use

- As the main implementation step in an autonomous development loop
- When a task has been clearly defined and selected by the task selection step
- Any workflow where the AI needs to make concrete code changes and verify them

## Implementation Approach

1. **Read context** — examine relevant source files, tests, and configuration to understand the current state
2. **Plan changes** — identify which files to create or modify, in what order
3. **Implement** — write the code, following existing patterns and conventions
4. **Test** — run the project's test command and check for failures
5. **Fix** — if tests fail, read the error output and fix the issues
6. **Commit** — stage the changes and commit with a descriptive message that references the task

## Inputs

- The selected task (from the task selection step)
- The project path and file structure
- Test and build commands
- Project conventions (coding style, framework patterns, test approach)

## Outputs

- Modified source files (committed to git)
- Test results (pass or fail with error output)
- A summary of what was changed and why
