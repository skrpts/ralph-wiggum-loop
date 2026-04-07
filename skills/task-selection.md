---
type: skill
id: task-selection
title: Task Selection
description: "Reads the progress tracker and selects the next incomplete task based on dependencies and priority"
tags: [Production, Developer, Planning]
connections:
  - target: llm-service
    type: runs_on
---

## Capability

Reads a progress tracking file (markdown checklist) and the current state of the codebase to determine which task to work on next. Considers:

- Which tasks are already complete (checked off in the progress file)
- Task dependencies — if task B depends on task A, A must be done first
- Natural ordering — foundational tasks (data models, configuration) before consumer tasks (API endpoints, UI)
- Previous iteration failures — if the last iteration attempted a task and the verifier flagged issues, this step reads the feedback and may re-select the same task or choose a different approach

## When to Use

- As the first step in each iteration of an autonomous development loop
- When you need structured task prioritisation from a flat checklist
- Any workflow where the AI must decide what to work on next based on current project state

## Inputs

- The progress tracker file (markdown checklist with task status)
- The project's file structure (to assess what already exists)
- Previous iteration feedback (if any) from the verifier

## Outputs

A structured selection:

- **Selected task:** the specific task to implement this iteration
- **Rationale:** why this task was chosen (dependency order, priority, retry)
- **Approach:** high-level plan for implementing the task
- **Dependencies:** what must be true before this task can succeed
