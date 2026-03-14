---
description: "Use when multiple LLM agents work concurrently in this repository. Do all implementation in isolated git worktrees and integrate only after syncing from main."
name: "Concurrent Agent Worktrees"
applyTo: "**"
---
# Concurrent Agent Worktrees

- Do all implementation in an isolated git worktree, for example under `.worktrees/` or an equivalent sibling worktree path, not in the shared main checkout.
- Create one branch per task inside that worktree and commit there while working.
- Treat the shared main checkout as read-only while implementation is in progress.
- Keep all edits, tests, conflict resolution, and commits inside the task worktree.
- When implementation is complete, finish the worktree lifecycle inside the worktree: pull the latest `main`, resolve conflicts there, run verification, push the integrated result to `origin/main`, and then delete the completed worktree.
- If direct pushes to `origin/main` are blocked by repository policy, stop and report the blocker instead of skipping the workflow.
- If a task cannot be isolated to its own worktree, stop and ask the user how they want to coordinate it.