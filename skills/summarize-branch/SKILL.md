---
name: summarize-branch
description: Use when Yuvraj names a branch and asks to summarize it, see what changed versus production, or verify that a branch's changes are correct.
---

# Summarize Branch

Summarize one branch against the production branch, flag errors, and report conflicts.

## Roles

- You own the summary and every judgment. Talk to Yuvraj yourself.
- The reviewer sub-agent (`claudem`, MiniMax in Claude Code, via cmux) only reads code and reports errors. Treat its output as raw input: verify each claimed error against the code before reporting it.

## Steps

1. Get the branch name from Yuvraj. If missing, ask. Confirm the production branch with Yuvraj (see `github-handler`), do not assume `main`.
2. `git fetch`, then check the branch exists locally or on the remote. Work read-only: never check out, commit to, or push either branch.
3. Compare against production using the merge base: `git diff --stat <prod>...<branch>`, `git log --oneline <prod>..<branch>`, and the full diff for the changed files. Read small diffs yourself. For large diffs, hand the reading to the reviewer (step 5).
4. Write a brief summary: what the branch does, grouped by area, with the notable files. Short, no per-line narration.
5. Verify correctness, only if Yuvraj asked for it (or asks later):
   - Analyze the changed code yourself for bugs, broken edge cases, missing error handling and inconsistencies with the surrounding code.
   - Also spawn a reviewer through cmux (see below) to check the changed code against the existing code it touches, and to report any errors it finds.
6. Check conflicts:
   - With production: `git merge-tree --write-tree <prod> <branch>` (exit code 1 means conflicts; list the files).
   - With other branches: if Yuvraj named any, or other open branches touch the same files (`git branch -r --no-merged <prod>` plus `git diff --name-only <prod>...<other>`), run the same merge-tree check and report real conflicts.
7. Report to Yuvraj: summary, errors found (file, line, why), conflicts (with which branch, which files). Say plainly if nothing was found.

## Reviewer via cmux

Same cmux rules as `codebase-map`: check cmux is running (`[ -n "$CMUX_SURFACE_ID" ]` or `cmux ping`). If not, do NOT spawn anywhere else. Ask Yuvraj to open cmux or confirm that you do the review yourself, and wait.

1. Save the diff outside the repo: `git diff <prod>...<branch> > ~/.orchestra/summarize-<branch-slug>.diff`. Write the task to `~/.orchestra/summarize-<branch-slug>.spec.md`: the diff path, the branch and production names, "read the diff and the existing code it touches, report bugs and risks as file:line plus reason, change nothing, end with a short list".
2. Launch in the current workspace (or one new workspace), not in a `wt-*` folder (that would trigger worker auto-reporting):
   `cmux new-split right --focus false --command "claude --settings ~/.claude/minimax.json 'Read ~/.orchestra/summarize-<branch-slug>.spec.md and do the task.'"`
3. Save the surface from `OK surface:<n> workspace:<m>` to `~/.orchestra/summarize-<branch-slug>.surface`. No timer is needed.
4. When it finishes, read its summary, close the pane (`cmux close-surface --surface $(cat ...surface)`), and remove the surface, spec and diff files. Run `cmux tree` to confirm no leftover panes. Never close your own surface.

## Rules

- Read-only on git: no checkout, commit, push, merge or rebase.
- Never report a reviewer-claimed error you have not verified.
- Never use the em dash in any output.
