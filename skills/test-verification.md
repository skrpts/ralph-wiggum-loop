---
type: skill
id: test-verification
title: Test Verification
description: "Verifies implementation correctness and determines whether the loop should continue, pass, or fail"
tags: [Production, Developer, Quality]
connections:
  - target: llm-service
    type: runs_on
---

## Capability

Acts as the **verifier** in the `until_pass` loop. After each implementation step, evaluates whether:

1. The current task was implemented correctly (tests pass, code is reasonable)
2. All tasks in the progress tracker are now complete
3. The overall project is in a healthy state (builds, no regressions)

Produces a structured verdict that controls the loop's next action.

## Evaluation Criteria

1. **Tests pass** — the test command exits with code 0, no test failures
2. **Build succeeds** — the build command (if specified) completes without errors
3. **Task complete** — the specific task that was attempted is genuinely done, not partially implemented
4. **No regressions** — previously passing tests still pass
5. **Progress updated** — the progress tracker reflects the completed task
6. **All done?** — are there remaining incomplete tasks in the progress tracker?

## When to Use

- As the verifier step in an autonomous development loop
- When you need a structured go/no-go decision after each implementation iteration
- Any quality gate where test results are the primary success criterion

## Inputs

- Test command output (exit code + stdout/stderr)
- Build command output (if applicable)
- The progress tracker file (to check overall completion)
- The implementation summary (what was changed)

## Outputs

A structured JSON verdict:

```json
{
  "status": "pass | continue | fail",
  "reason": "Specific explanation of the verdict",
  "instructions": "What to do next (only when status is continue)"
}
```

### Verdict meanings

| Status | When | Effect |
|--------|------|--------|
| `pass` | All tasks complete, all tests passing | Loop ends successfully |
| `continue` | Current task done but more remain, or task needs another attempt | Loop runs again with instructions for next iteration |
| `fail` | Unrecoverable problem — broken dependencies, impossible task, repeated failures on same issue | Loop stops with error |

### Continue instructions

When returning `continue`, the instructions field should specify:

- Whether to retry the same task or move to the next one
- What specific issue needs addressing (if retrying)
- Any context the next iteration needs to know
