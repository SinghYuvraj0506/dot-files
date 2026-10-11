---
name: cmux
description: Use when the user mentions cmux, or when you need to control the terminal multiplexer app from a shell - open workspaces, splits, tabs or browser panes, send keys or text to another terminal surface, read another pane's screen, take screenshots, set sidebar status or progress, or send notifications.
---

# cmux

## Overview

`cmux` is a CLI that controls the cmux app over a Unix socket. Inside a cmux terminal, `CMUX_WORKSPACE_ID` and `CMUX_SURFACE_ID` are set, so most commands default to the caller's own workspace and surface.

For anything not covered here, run `cmux help <group>` (no socket needed). Groups: `start agents navigate inspect customize automation browser remote diagnostics`. Official agent guide: `cmux guide` or `cmux --skill`.

## Targets

Windows, workspaces, panes and surfaces take a UUID, a short ref (`workspace:2`, `pane:3`, `surface:4`) or an index. Output defaults to refs; add `--id-format uuids|both` for UUIDs. Use `cmux tree` or `cmux identify` to discover refs.

## Quick Reference

| Goal | Command |
|---|---|
| Open dir in new workspace | `cmux <path>` or `cmux new-workspace --cwd <path> --name <t> [--command <c>]` |
| Where am I / what exists | `cmux identify`, `cmux tree [--all]`, `list-workspaces`, `list-panes`, `list-panels` |
| Split | `cmux new-split <left\|right\|up\|down> [--command <c>]` |
| New pane / tab | `cmux new-pane --type terminal\|browser`, `cmux new-surface --type ...` |
| Close | `close-surface`, `close-workspace --workspace <ref>` |
| Rename | `rename-workspace <t>`, `rename-tab <t>` |
| Type into a surface | `cmux send --surface <ref> "text"` (add `\n` or follow with `send-key enter`) |
| Press a key | `cmux send-key --surface <ref> <key>` |
| Paste / submit | `cmux paste --surface <ref> --submit "text"` |
| Read another pane | `cmux read-screen --surface <ref> [--scrollback] [--lines <n>]` |
| Screenshot / record | `cmux shot --out <path>`, `cmux record start --gif --out <path>` / `record stop` |
| Notify user | `cmux notify --title <t> --body <b>` |
| Sidebar status | `set-status <key> <value>`, `set-progress 0.5 --label <t>`, `log <msg>` |
| Show a diff | `cmux diff --source unstaged\|staged\|branch\|last-turn` |
| Show markdown | `cmux markdown open <path>` |
| Message another agent | `cmux agent message <target> <text>`, `cmux agent inbox` |
| Block until signalled | `cmux wait-for [-S] <name> [--timeout <s>]` |
| Reload config | `cmux reload-config` (Ghostty config and `~/.config/cmux/cmux.json`) |

## Browser pane

`cmux browser [--surface <ref>] <sub>`: `open <url>`, `goto <url>`, `snapshot -i`, `click|fill|type <selector> ...`, `get text|html|title|url`, `wait --selector|--text|--load-state`, `eval <js>`, `screenshot --out <path>`, `console list`, `errors list`. Add `--snapshot-after` to actions to see the result.

## Workflow patterns

- Run a long task in a side pane: `cmux new-split right --command "<cmd>"`, then `cmux read-screen --surface <ref>` to check output.
- Drive a second agent or REPL: `cmux send --surface <ref> "prompt"` then `cmux send-key --surface <ref> enter`.
- Tell the user a task is done: `cmux notify --title "Done" --body "..."`.

## Settings and config

Back up any existing `cmux.json` to a timestamped `.bak` before editing. Use `cmux settings path` and `cmux docs settings` for the schema. Prefer Ghostty config (`~/.config/ghostty/config`) for terminal behavior Ghostty already supports (font, theme, transparency, keybinds).

## Common mistakes

- Targeting by index after panes change: re-run `cmux tree` or use refs/UUIDs.
- Sending text without a newline or `enter` key, so it never executes.
- Editing `cmux.json` without a backup or without `cmux reload-config` afterwards.
- Running outside a cmux terminal: no default workspace/surface, so pass `--workspace`/`--surface` explicitly.
