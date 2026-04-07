---
type: asset
id: loop-runner-script
title: Loop Runner Script
description: "Reference bash script for running the Ralph Wiggum loop with Claude Code CLI"
tags: [Production, Developer, Infrastructure]
connections: []
metadata:
  asset_type: "reference"
  format: "markdown"
---

# Loop Runner Script

This asset provides the bash loop that drives the Ralph Wiggum pattern when running outside of skrptiq's built-in loop execution. Use this when running Claude Code directly from the terminal.

## Basic Loop

```bash
#!/bin/bash
# ralph-loop.sh — autonomous development loop for Claude Code
#
# Usage: ./ralph-loop.sh [max-iterations]

MAX_ITERATIONS="${1:-10}"
PROMPT_FILE="PROMPT.md"
PROGRESS_FILE="PROGRESS.md"
ITERATION=0

echo "Ralph Wiggum loop starting (max $MAX_ITERATIONS iterations)"

while [ "$ITERATION" -lt "$MAX_ITERATIONS" ]; do
  ITERATION=$((ITERATION + 1))
  echo ""
  echo "=== Iteration $ITERATION / $MAX_ITERATIONS ==="
  echo ""

  # Run Claude Code with the prompt file, piping it as stdin
  # The --print flag runs non-interactively and exits when done
  cat "$PROMPT_FILE" | claude --print

  # Check if all tasks are complete
  if grep -q "^- \[ \]" "$PROGRESS_FILE" 2>/dev/null; then
    echo "Tasks remain — continuing loop"
  else
    echo "All tasks complete — exiting loop"
    break
  fi
done

echo ""
echo "Loop finished after $ITERATION iterations"
```

## With Cost Controls

```bash
#!/bin/bash
# ralph-loop-safe.sh — with rate limiting and cost controls

MAX_ITERATIONS="${1:-10}"
COOLDOWN_SECONDS=30
PROMPT_FILE="PROMPT.md"
PROGRESS_FILE="PROGRESS.md"
ITERATION=0
CONSECUTIVE_FAILURES=0
MAX_CONSECUTIVE_FAILURES=3

while [ "$ITERATION" -lt "$MAX_ITERATIONS" ]; do
  ITERATION=$((ITERATION + 1))
  echo "=== Iteration $ITERATION / $MAX_ITERATIONS ==="

  # Run with timeout (15 minutes max per iteration)
  timeout 900 bash -c "cat $PROMPT_FILE | claude --print"
  EXIT_CODE=$?

  # Check for timeout
  if [ "$EXIT_CODE" -eq 124 ]; then
    echo "WARNING: Iteration timed out after 15 minutes"
    CONSECUTIVE_FAILURES=$((CONSECUTIVE_FAILURES + 1))
  elif [ "$EXIT_CODE" -ne 0 ]; then
    echo "WARNING: Claude exited with code $EXIT_CODE"
    CONSECUTIVE_FAILURES=$((CONSECUTIVE_FAILURES + 1))
  else
    CONSECUTIVE_FAILURES=0
  fi

  # Circuit breaker
  if [ "$CONSECUTIVE_FAILURES" -ge "$MAX_CONSECUTIVE_FAILURES" ]; then
    echo "ERROR: $MAX_CONSECUTIVE_FAILURES consecutive failures — stopping loop"
    break
  fi

  # Check completion
  if ! grep -q "^- \[ \]" "$PROGRESS_FILE" 2>/dev/null; then
    echo "All tasks complete"
    break
  fi

  # Cooldown between iterations
  echo "Cooling down for ${COOLDOWN_SECONDS}s..."
  sleep "$COOLDOWN_SECONDS"
done

echo "Loop finished: $ITERATION iterations, $CONSECUTIVE_FAILURES recent failures"
```

## Notes

- The `--print` flag (or `-p`) runs Claude Code non-interactively. It reads the prompt from stdin, executes, and exits.
- Each iteration gets a fresh context window — this is the key advantage over running in a single long session.
- The progress file on disk is the handoff mechanism. Claude reads it at the start of each iteration to know what's done and what remains.
- Set `MAX_ITERATIONS` conservatively. Each iteration is a full AI session. Start with 5 and increase once you see the pattern working for your project.
- The cooldown in the safe version prevents rate limiting and gives you time to interrupt if needed (`Ctrl+C` during the sleep).
