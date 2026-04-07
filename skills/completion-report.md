---
type: skill
id: completion-report
title: Completion Report
description: "Produces a summary of what the autonomous loop accomplished across all iterations"
tags: [Production, Developer, Reporting]
connections:
  - target: llm-service
    type: runs_on
---

## Capability

Runs after the development loop ends (whether by passing, reaching max iterations, or failing). Produces a structured report covering:

- Which tasks were completed and in which iteration
- Which tasks remain incomplete (if any)
- Total iterations run and the final loop status
- Key decisions and trade-offs made during implementation
- Any issues or technical debt introduced

## When to Use

- As the post-loop step after an autonomous development cycle
- When you need a record of what an autonomous agent accomplished
- For handoff — giving a human developer a clear picture of the current state

## Inputs

- The final progress tracker file
- The loop status and iteration count (from `{{loop.dev-cycle.status}}` and `{{loop.dev-cycle.iterations}}`)
- The final iteration's output (from `{{loop.dev-cycle.final}}`)
- Git log of commits made during the loop

## Outputs

A structured completion report in markdown:

- **Summary** — one paragraph overview of what was accomplished
- **Task status table** — each task with its completion status and the iteration it was done in
- **Iteration log** — brief notes on what each iteration did
- **Remaining work** — any incomplete tasks with notes on why they weren't finished
- **Technical notes** — anything a human developer should review or be aware of
