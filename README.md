# dot-files

Yuvraj's agent setup: shared instructions, skills, and the multi-agent orchestration system (UV Orchestrator) built on cmux.

## What lives in this repo

| Path | Purpose | Linked from |
|---|---|---|
| `AGENTS.md` | Global agent instructions (also the commit co-author rule) | `~/.claude/CLAUDE.md` (symlink) |
| `skills/` | All custom skills: `cmux`, `codebase-map`, `github-handler`, `initialize-agents-md`, `uv-orchestrator` | `~/.claude/skills` (symlink) |
| `bin/orch-timer` | 5-minute check-in timer for a worker | `~/.local/bin/orch-timer` (symlink) |
| `bin/worker-peek` | Progress snapshot and comparison for a worker | `~/.local/bin/worker-peek` (symlink) |

Always edit the files here, never the symlink targets.

## UV Orchestrator: how it works

- The orchestrator is a plain `claude` session in a cmux pane, started from the main repo folder.
- Workers run in git worktrees named `../wt-<task>`. Any session whose folder starts with `wt-` is treated as a worker.
- Workers never get polled. They report into the orchestrator's pane by typing a `[worker:wt-<task>] ...` message, so waiting costs zero tokens.
- Two kinds of report:
  - `done` and `needs permission`: sent automatically by hooks when the worker stops or hits a permission prompt.
  - `5-min check-in`: sent by `orch-timer` while the worker is still running. The orchestrator runs `worker-peek` and only speaks up if there is no progress, a possible loop, or the wrong direction.
- Full behaviour and the orchestrator's rules are in `skills/uv-orchestrator/SKILL.md`. That file is the source of truth.

## Implemented

Inside this repo (versioned):
- `skills/uv-orchestrator/SKILL.md`: session start (`orch_surface`), Dispatch, Waiting, Check-ins sections. All polling and screen-reading rules removed.
- `bin/orch-timer <task>` and `bin/orch-timer --cancel <task>`:
  - Loop every `ORCH_TIMER_INTERVAL` seconds (default 300), PID in `~/.orchestra/<task>.timerpid`, restarting kills the old timer.
  - Exits silently once `~/.orchestra/<task>.md` is newer than `<task>.timer` (worker reported).
- `bin/worker-peek <task>`:
  - Prints a PROGRESS line, elapsed time, turn count, changed files and the last 2 assistant messages.
  - Saves a snapshot to `~/.orchestra/<task>.lastpeek` and compares with the previous one.
  - Flags `POSSIBLE LOOP` when turns go up but no files change.
  - Counts untracked files as progress. Works for Claude and MiniMax workers only (reads `~/.claude/projects` transcripts).

Outside this repo (NOT versioned, must exist on the machine):
- `~/.claude/hooks/worker-report.sh`: Claude worker hook for Stop and Notification (permission only) events. Writes the last assistant message to `~/.orchestra/<task>.md` and messages the orchestrator.
- `~/.claude/settings.json`: registers `worker-report.sh` under `Stop` (next to the existing notifier hooks) and a new `Notification` hook.
- `~/.config/cmux/report-to-orch.sh`: catch-all for non-Claude workers (Codex, Gemini CLI, ...). Skips `agent.kind == claude`, non-`wt-*` folders, and pending states.
- `~/.config/cmux/cmux.json`: `notifications.hooks` entry `report-to-orch` with `timeoutSeconds` 10. The file is JSONC, keep its comments.
- `~/.claude/minimax.json`: MiniMax env block only. It must NOT contain worker-report hooks or they fire twice.
- `.zshrc` alias `claudem='claude --settings ~/.claude/minimax.json'` (there is no `mm` command).

## Checklist to get it working on a fresh machine or after a reset

1. `jq` installed, `cmux capabilities` works, `~/.local/bin` is on PATH.
2. Symlinks exist:
   - `~/.claude/CLAUDE.md` -> `AGENTS.md`
   - `~/.claude/skills` -> `skills/`
   - `~/.local/bin/orch-timer` -> `bin/orch-timer`
   - `~/.local/bin/worker-peek` -> `bin/worker-peek`
3. The four outside-repo files above exist, scripts are `chmod +x`, and `cmux reload-config` was run after editing `cmux.json`.
4. Start the orchestrator from a cmux pane. It must run `mkdir -p ~/.orchestra && echo $CMUX_SURFACE_ID > ~/.orchestra/orch_surface` (the skill does this at step 0). If `CMUX_SURFACE_ID` is empty the session is not in a cmux pane and nothing will report back.

## Known gaps and to-do

- The hook scripts and the `settings.json` / `cmux.json` edits live outside the repo. Consider moving `worker-report.sh` and `report-to-orch.sh` here and symlinking them, so a reset does not lose them.
- Not yet tested in a real pane: the original `$CMUX_SURFACE_ID` path and `report-to-orch.sh` with a real Codex or Gemini worker. The timer test used a scratch workspace as a stand-in orchestrator.
- The "done" report appeared twice in the timer test (cause not investigated; maybe two Stop events or a duplicate hook).
- The skill's nudge and stop sequence (`cmux send`, escape, "Stop and summarize...") has not been exercised against a stuck worker.
- `worker-peek` cannot inspect non-Claude workers (no transcript). For those the orchestrator uses `git diff --stat` in the worktree.
- `.orchestra` files per task: `<task>.spec.md`, `.surface`, `.md`, `.timer`, `.timerpid`, `.lastpeek`. Clean them up after a task is finished.
