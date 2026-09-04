---
name: daily-note
description: Add to today's daily note in the user's Obsidian vault, creating it if missing, then commit and push. Use when the user says "daily note", "today's note", "log this", "journal", or "remember that today ...". Also used by routines that write a daily summary.
---

# Daily note

## Before you write

1. Read the config. Grok Bot: `/workspace/obsidian-vault/config.md`. Laptop: `~/.obsidian-vault-bot/config.md`. If missing, run `vault-setup` first.
2. If `mode: git`, run `git -C <vault> pull --rebase --quiet`. Stop on error.

## Work out the path

- Today's date in the config `timezone`. Use `TZ=<timezone> date +%F` to be sure. Do not guess the date.
- Path: `<vault>/<daily_folder>/<date in daily_format>.md`. Example: `Daily/2026-09-04.md`.
- If the user names another day ("yesterday", "on Monday"), use that date. Say which file you used.

## Create if missing

First make sure the folder exists: `mkdir -p "<vault>/<daily_folder>"`. Git does not
keep empty folders, so a fresh clone may not have it. Then write:

```markdown
---
date: <YYYY-MM-DD>
tags: [daily]
---

## Log

```

No `#` title line. Obsidian shows the filename as the title already.

Then treat it as an existing note.

## Append

- Read the file first.
- Append under `## Log` if it exists, else at the end.
- Format: `- <HH:mm> <text>` in the config timezone.
- For a summary from a routine, use a heading instead: `## Summary` followed by three to six bullets.
- Never rewrite earlier lines. Never remove anything.

## Commit and push (git mode only)

```bash
git -C <vault> add "<path>"
git -C <vault> commit -m "note: daily <date>"
git -C <vault> push
```

If push is rejected: `git -C <vault> pull --rebase` once, then push again. If it still fails, stop and show the error. Do not force push.

## Tell the user

One line: `Added to Daily/2026-09-04.md under Log.` Say "created" if the file is new.

## Rules

- Never delete or rename files.
- Never touch `.obsidian/`, `.git/`, or binary files.
- One commit per request. A routine that runs once a day makes one commit.
