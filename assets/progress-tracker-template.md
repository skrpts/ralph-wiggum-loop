---
type: asset
id: progress-tracker-template
title: Progress Tracker Template
description: "Template for the progress file that tracks task completion across loop iterations"
tags: [Production, Developer, Template]
connections: []
metadata:
  asset_type: "template"
  format: "markdown"
---

# Progress Tracker

> This file is the handoff mechanism between loop iterations. Each iteration reads it to determine what to work on next, and updates it after completing a task. Do not delete or restructure this file while the loop is running.

## Project

- **Name:** [Project name]
- **Started:** [Date/time the loop started]
- **Max iterations:** [Configured limit]

## Tasks

- [ ] [Task 1 — clear, specific description]
- [ ] [Task 2 — clear, specific description]
- [ ] [Task 3 — clear, specific description]
- [ ] [Task 4 — clear, specific description]

## Iteration Log

### Iteration 1

- **Task:** [Selected task]
- **Result:** [Done / Failed / Partial]
- **Commit:** [hash]
- **Notes:** [Brief summary of what happened]

---

## Writing Good Tasks

Tasks should be:

- **Specific.** "Add JWT authentication middleware to the Express app" — not "add auth".
- **Testable.** The verifier needs a way to confirm the task is done. If tests exist, they should cover the task's behaviour.
- **Independent where possible.** Tasks that depend on each other should be listed in dependency order, but each task should be completable in a single iteration.
- **Right-sized.** One task = one meaningful unit of work. A task that takes 3 iterations is too big. A task that's done in 10 lines is too small — batch it with related work.

## Task States

| Marker | Meaning |
|--------|---------|
| `- [ ]` | Not started |
| `- [x]` | Complete — tests pass, code committed |
| `- [!]` | Failed — attempted but could not complete (see notes) |
| `- [~]` | In progress — started but not verified (loop may have been interrupted) |
