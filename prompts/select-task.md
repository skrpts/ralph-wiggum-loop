---
type: prompt
id: select-task
title: Select Task
description: "Reads the progress tracker and selects the next task to implement"
tags: [Production, Developer, Planning]
inputs:
  project_path:
    label: "Project Path"
    description: "Path to the project root directory"
    example: "/home/user/my-app"
    required: true
    type: text
  test_command:
    label: "Test Command"
    description: "The command to run the test suite"
    example: "npm test"
    required: true
    type: text
  build_command:
    label: "Build Command"
    description: "The command to build the project"
    example: "npm run build"
    required: true
    type: text
  conventions:
    label: "Coding Conventions"
    description: "Project-specific coding conventions and style rules"
    example: "TypeScript strict, ESM imports, vitest for tests"
    required: true
    type: text
  task_list:
    label: "Task List"
    description: "Tasks to complete — as a markdown checklist or description"
    example: "- [ ] Add input validation\n- [ ] Write integration tests\n- [ ] Update API docs"
    required: true
    type: text
connections:
  - target: task-selection
    type: derived_from
metadata:
  output_format: markdown
  prompt_type: core
---

## Purpose

Drives the task selection skill. Reads project state and the progress tracker to decide what to work on next.

## Prompt

You are an autonomous development agent starting a new iteration. Your job is to read the current state and select the next task to implement.

### Project

- **Path:** {{input.project_path}}
- **Test command:** {{input.test_command}}
- **Build command:** {{input.build_command}}
- **Conventions:** {{input.conventions}}

### Task List

{{input.task_list}}

### Previous Iteration Feedback

{{loop.lastReview}}

### Instructions

1. Read the progress tracker file at the project root (if it exists). If this is the first iteration, create one from the task list above.
2. Identify which tasks are already complete (marked with `[x]`).
3. Select the next incomplete task. Prefer tasks whose dependencies are already satisfied.
4. If the previous iteration's feedback says to retry a task, select that task and note the feedback.

### Output Format

Respond with:

- **Selected task:** [the task description]
- **Status:** [new | retry]
- **Rationale:** [why this task is next — dependency order, priority, retry reason]
- **Approach:** [2-3 sentence plan for implementation]
- **Files to examine first:** [list of files to read before starting]
