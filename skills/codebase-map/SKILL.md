---
name: codebase-map
description: Use when creating or refreshing a codebase map (per-module docs in docs/AI) for a project, or when a project has no docs/AI map yet.
---

# Codebase Map

One Markdown file per module in the project's `docs/AI/`, indexed from the project's `AGENTS.md`.

## Roles

- You own every decision: which modules, what to ask Yuvraj, what to accept from a sub-agent, what goes in the docs. Talk to Yuvraj yourself and judge all input (his and the sub-agents') before acting.
- You may read code yourself when it helps you decide or verify. Light reading is fine.
- Delegate heavy work (reading many files, large diffs, per-module research) to sub-agents. Sub-agents run on a low-cost model via cmux with `claudem` (MiniMax in Claude Code), so they exist to save your tokens, not to decide anything.
- Sub-agent output is raw input. Check it for gaps or errors, spot-check key claims against the code, and re-run or correct it where it falls short.

## cmux precondition

Sub-agents are spawned only through cmux. Before spawning any, check that cmux is running (e.g. `[ -n "$CMUX_SURFACE_ID" ]` or `cmux ping`). If cmux is not open, do NOT spawn sub-agents anywhere else (no in-process Agent tool, no other terminal). Ask Yuvraj to either open cmux or explicitly confirm that you may do the delegated work yourself inside this session, and wait for his answer. Do not proceed with any delegated step until he confirms.

Spawn every sub-agent as a pane (`cmux new-split` / `cmux new-pane`) inside one single workspace. Use the current workspace, or create one new workspace (`cmux new-workspace`) if needed, but keep all sub-agents together in that one workspace. Never spread them across multiple workspaces.

## Cleanup

Every sub-agent pane must be closed once its job is done. Never leave finished panes open.

- Right after spawning, save the pane's surface (from the `OK surface:<n> workspace:<m>` output) to a scratch file, e.g. `echo surface:<n> > /tmp/codebase-map-<module>.surface`.
- As soon as you have read a sub-agent's summary and need no follow-up, close it: `cmux close-surface --surface $(cat /tmp/codebase-map-<module>.surface)`, then remove the file. If you send a follow-up, keep the pane open until that finishes.
- Never close your own surface.
- Before telling Yuvraj the map is done, run `cmux tree` and confirm none of your sub-agent panes remain. Close leftovers. If you created a workspace via `cmux new-workspace` for them, close it with `cmux close-workspace --workspace <ref>`.

## Create (no map exists)

1. Ask Yuvraj which modules to document. Suggest modules from the folder structure.
2. For each module, ask Yuvraj what it does, how it does it, and what it contains.
3. Decide the scope per module. For any module larger than a few files, spawn one explorer sub-agent per module via cmux, running `claudem`. Point it at the file paths Yuvraj gave (or have it research the module), with Yuvraj's description. It collects how everything works and returns a summary. Close its pane once you have the summary (see Cleanup). Small modules you may read yourself.
4. Review the summary critically, verify what looks doubtful, then write `docs/AI/<module>.md` using the template below. Keep it short and basic.
5. Add an entry to `AGENTS.md`: the doc path, one line on what it covers, and the current commit hash (`git rev-parse HEAD`).

## Refresh (map exists)

1. Confirm the production branch with Yuvraj (see `github-handler`), then check recent merges on it since the stored hashes (`git log --merges <hash>..<production branch>`).
2. Decide which modules look affected, and suggest to Yuvraj what merged and which modules to refresh or newly create. Wait for approval before changing anything.
3. Read the commit hash stored in `AGENTS.md` for each approved module.
4. Run `git diff <hash>..<production branch> -- <module paths>` and for large diffs have a `claudem` sub-agent summarize the changes (read small diffs yourself). Verify the summary, close the sub-agent pane (see Cleanup), then update only the affected docs.
5. Write the new production branch commit hash back into `AGENTS.md`.

## Module template

```markdown
# <Module name>

## Purpose
One or two lines: what it does.

## How it works
Short flow: main steps, data in and out.

## Files
- `path/to/file` - role

## Notes
Key constraints, gotchas, dependencies on other modules.
```

## Rules

- Keep each doc concise. Detail belongs in code, not here.
- Never document a module Yuvraj did not list or approve.
- Never paste sub-agent output into a doc unreviewed.
- Always update `AGENTS.md` after creating or refreshing a doc.
- Close every sub-agent pane you spawned when its job is done (see Cleanup).
