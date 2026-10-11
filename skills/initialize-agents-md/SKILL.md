---
name: initialize-agents-md
description: Use when a project has no AGENTS.md, or Yuvraj asks to set up or sync agent instruction files (AGENTS.md, CLAUDE.md, GEMINI.md) in a project.
---

# Initialize AGENTS.md

`AGENTS.md` is the single source of truth. Other agent files only import it.

## Steps

1. If `AGENTS.md` already exists, read it and ask Yuvraj before changing anything.
2. Ask Yuvraj for:
   - the project name, a short description, and the production branch (confirm it, not always `main`)
   - how the project is run locally
   - how it should be tested (record his answer as the testing criteria)
   - whether to use existing database/Redis services: check for Docker Compose files (`docker-compose.yml`, `compose.yaml`) and offer any DB or Redis service found
3. Create `AGENTS.md` from the template below.
4. Create the other agent files containing only an import of `AGENTS.md`:
   - `CLAUDE.md`: `@AGENTS.md`
   - `GEMINI.md`: `@AGENTS.md`
   Ask Yuvraj if any other agent files are needed.
5. Verify every other agent file imports `AGENTS.md`, so updating it updates them all. Re-check this whenever `AGENTS.md` is edited.

## AGENTS.md template

```markdown
# <Project name>

<One or two lines: what the project is.>

## Production branch
`<branch>` is live on production. Never change or push to it directly.

## Workflow
For every new feature: create a new branch, then merge via pull request.

## Local run
<Exactly how Yuvraj runs the project locally. Always run it this way.>

## Testing criteria
<How Yuvraj wants this repo tested, in his words.>

## Database
<Compose service to use (DB/Redis) if Yuvraj approved one, otherwise how to get a database.>

## Codebase map
See `docs/AI/` (built with the `codebase-map` skill).
```

## Rules

- Keep `AGENTS.md` short. Never copy its content into the other files.
- Never overwrite an existing agent file without asking.
- Git/GitHub actions follow the `github-handler` skill.
