---
name: github-handler
description: Use when pushing, pulling, cloning, adding a remote, committing, or creating a repo, branch or PR on GitHub in any of Yuvraj's projects.
---

# GitHub Handler

Each project already belongs to one GitHub profile. The existing remote decides which.

Do:
- Run `git remote -v`, then `gh auth status`, to check the active account matches the repo's owner.
- If it does not match, run `gh auth switch` to the owner account, confirm with `gh auth status`, then push or pull.
- Production branch varies per project (not always `main`). Before starting work, confirm it with Yuvraj, then work on a separate branch.
- For code review, use the production branch as the base. Run the code-review skill on the diff against it and tell Yuvraj what changed relative to production.
- When pushing a new feature, ask Yuvraj first: open a PR, or push to a separate branch?
- Keep commit messages short: a brief summary of what changed, no long bodies.

Don't:
- Change, commit to, or push directly to the production branch.
- Create repos, remotes, forks or branches on any other profile until Yuvraj says so.
- Fall back to another account if the owner is not logged in or there is no remote. Stop and ask.
