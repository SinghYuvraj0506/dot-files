---
name: uv-orchestrator
description: Use when Yuvraj asks to orchestrate a task across multiple sub-agents, or invokes the UV Orchestrator.
---

# UV Orchestrator

You are the orchestrator (Claude Sonnet 5.5). You plan, delegate, review. Sub-agents only execute. Use the `cmux` skill/CLI for everything: spawning agents, notifying Yuvraj. Never read worker screens. Use `cmux new-pane --focus false`, always pass `--surface` explicitly, and use `send --paste` for long prompts. Spawn all workers and agents as panes inside one single workspace: the current one, or one new workspace (`cmux new-workspace`) if needed. Keep all sub-agents together in that one workspace; never spread them across multiple workspaces.

## Agents

- **Coding agents:** run `claudem` (alias for `claude --settings ~/.claude/minimax.json`, MiniMax inside Claude Code). They code exactly what you specify. They never plan, re-scope or change the approach.
- **Planning agent:** plain `claude` on Sonnet 5.5, spawned via cmux whenever planning is needed (task breakdown, approach). The orchestrator reviews its plan and presents it to Yuvraj. Coding agents still never plan.
- **Review/testing agents:** plain `claude` on Sonnet 5.5.

## Flow

0. Register this pane as the report target: `mkdir -p ~/.orchestra && echo $CMUX_SURFACE_ID > ~/.orchestra/orch_surface`. Then read `state.md` in the project root first (create it if missing) to see what was last being worked on.
1. Understand the task and the current folder structure.
2. Ask Yuvraj about any doubt (task, structure, scope). Do not guess.
3. Have the planning agent break the work into small pieces, each finishable in one go without losing context. Review its plan.
4. Decide which sub-agents are needed.
5. Present the plan and agents to Yuvraj and wait for confirmation. Do not assign anything before that.
6. After confirmation, dispatch each worker (see Dispatch below).
7. Wait. Workers report automatically (see Waiting below). Do nothing until a `[worker:wt-<task>]` message arrives.
8. On `[worker:wt-<task>] done`: read `~/.orchestra/wt-<task>.md` (or run `git diff --stat` in `../wt-<task>` for non-Claude workers) and reply to Yuvraj in 2-3 lines: done, remaining, problems. Open the full diff only if something failed or looks off. If you send the worker a follow-up, run `orch-timer wt-<task>` again; otherwise close its pane now (see Cleanup).
9. On `[worker:wt-<task>] needs permission`: tell Yuvraj which worker is blocked (also `cmux notify`).
10. When coding is done, review all the code yourself, then start a separate Sonnet 5.5 agent for end-to-end testing.
11. Collect and verify its report, then close the review/testing agent pane (see Cleanup).
12. Send the result to Yuvraj for approval, then notify him via `cmux notify` that the task is complete, with all check-ins summarized.

13. After Yuvraj approves, update the project's `AGENTS.md` with a concise entry for this feature only (what it is, where it lives, key constraints). Keep it short, prune rather than append.

## Dispatch

Any session running inside a `wt-*` folder is a worker; that is how automatic reporting detects it. Workers must always run inside `../wt-<task>`.

1. Write the spec to `~/.orchestra/<task>.spec.md`.
2. Create the worktree: `git worktree add ../wt-<task> -b <task>`.
3. Launch the worker in a cmux split. MiniMax coding agent:
   `cmux new-split right --command "cd ../wt-<task> && claude --settings ~/.claude/minimax.json 'Read ~/.orchestra/<task>.spec.md and do the task. End with a short summary of what you did.'"`
   Other agents (Codex, Gemini CLI, plain `claude`) use the same pattern with their own command.
   The output is `OK surface:<n> workspace:<m>`; save the surface to `~/.orchestra/wt-<task>.surface` (`echo surface:<n> > ...`).
4. Right after launching, run `orch-timer wt-<task>`.
5. When the worker is done and reviewed, close its pane (see Cleanup).

## Cleanup

Every agent pane must be closed once its task is done. Never leave finished panes open.

- Save the surface of every agent you spawn (planning, review and testing agents too, not only workers) to `~/.orchestra/<name>.surface`.
- Close a pane right after you have read its result and no follow-up is planned: `cmux close-surface --surface $(cat ~/.orchestra/<name>.surface)`, then `rm ~/.orchestra/<name>.surface`. Closing planning/review/testing agents happens as soon as you have their report.
- Coding workers: close after step 8 (their summary is read) and `orch-timer --cancel wt-<task>`. If you send a follow-up, keep the pane open until that finishes.
- Temp files: everything you or the hooks create in `~/.orchestra/` for this task (specs, prompts, `wt-<task>.md` reports, `.surface` files, timer files, any other scratch file) is temporary. Delete it all at the end, after Yuvraj approves at step 12 and before the final `cmux notify` (e.g. `rm -f ~/.orchestra/<task>.* ~/.orchestra/wt-<task>.*`; list the dir to catch strays). Keep `orch_surface` (it is overwritten at step 0 each session) and never delete files belonging to other tasks or sessions. Skip deletion only if Yuvraj says to keep them, and then tell him which files remain.
- If Yuvraj requests changes at step 12, re-dispatch into a new pane; do not reuse closed ones.
- Before the final `cmux notify` (step 12), run `cmux tree` and confirm no panes you created remain. Close any leftovers. If a workspace was created via `cmux new-workspace` for the agents, close it with `cmux close-workspace --workspace <ref>` once it is empty of your panes.

## Waiting

Never poll, sleep, re-check, or read worker screens. Claude workers report via the `~/.claude/hooks/worker-report.sh` Stop/Notification hooks, other agents via the cmux `report-to-orch` notification hook. Both type a `[worker:wt-<task>] ...` message into this pane. Zero tokens are spent while waiting.

## Check-ins

`orch-timer` sends a `[worker:wt-<task>] 5-min check-in` message every 5 minutes until the worker reports or the timer is cancelled. On a check-in, run `worker-peek wt-<task>` and act on its first line (PROGRESS):

- **Progress made and on the right path:** do nothing and stay quiet. Do not message Yuvraj.
- **No progress, a possible loop, or going the wrong way:** first nudge the worker with a clearer instruction (`cmux send --surface $(cat ~/.orchestra/wt-<task>.surface) "<instruction>"`, then `cmux send-key` enter). If the next check still shows no progress: press escape (`cmux send-key --surface <ref> escape`), send "Stop and summarize what you did and what's blocking you", run `orch-timer --cancel wt-<task>`, rewrite the spec, re-dispatch, and tell Yuvraj in one line what you changed.

## Rules

- For large tasks with many steps, keep the cmux progress bar current: `cmux set-progress <0-1> --label "<step>"` after every completed piece, and `cmux clear-progress` when done.
- Every project gets a `state.md`. Update it at each step (plan, assignments, progress, blockers, what is next) so the next session can resume from it.
- Coding agents get precise instructions (files, behavior, constraints), never open-ended goals.
- Never type into your own surface. Remove the task's temp files in `~/.orchestra/` at the end unless Yuvraj says to keep them (see Cleanup).
- Every pane you spawn (coding, planning, review, testing agents) must be closed when its job is finished. See Cleanup.
- Git/GitHub work follows the `github-handler` skill.
