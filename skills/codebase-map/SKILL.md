---
name: codebase-map
description: Use when creating or refreshing a codebase map (per-module docs in docs/AI) for a project, or when a project has no docs/AI map yet.
---

# Codebase Map

One Markdown file per module in the project's `docs/AI/`, indexed from the project's `AGENTS.md`.

## Create (no map exists)

1. Ask Yuvraj which modules to document. Suggest modules from the folder structure.
2. For each module, ask Yuvraj what it does, how it does it, and what it contains.
3. Do not explore the code yourself. Spawn one explorer sub-agent per module via cmux, running `claudem` (MiniMax in Claude Code). Point it at the file paths Yuvraj gave (or have it research the module), with Yuvraj's description. It collects how everything works and returns a summary.
4. Read the sub-agent's summary, then write `docs/AI/<module>.md` using the template below. Keep it short and basic.
5. Add an entry to `AGENTS.md`: the doc path, one line on what it covers, and the current commit hash (`git rev-parse HEAD`).

## Refresh (map exists)

1. Confirm the production branch with Yuvraj (see `github-handler`), then check recent merges on it since the stored hashes (`git log --merges <hash>..<production branch>`).
2. Suggest to Yuvraj what merged and which modules to refresh or newly create. Wait for approval before changing anything.
3. Read the commit hash stored in `AGENTS.md` for each approved module.
4. Run `git diff <hash>..<production branch> -- <module paths>` and have a `claudem` sub-agent summarize the changes. Update only the affected docs from its summary.
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
- Always update `AGENTS.md` after creating or refreshing a doc.
