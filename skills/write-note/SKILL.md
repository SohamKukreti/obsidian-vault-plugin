---
name: write-note
description: Create a new note in the user's Obsidian vault with frontmatter and wikilinks, then commit and push it. Use when the user says "file this", "save this as a note", "write a note about", or "put this in my vault". If the note already exists, hand off to append-note.
---

# Write a new note

## Before you write

1. Read the config. Grok Bot: `/workspace/obsidian-vault/config.md`. Laptop: `~/.obsidian-vault-bot/config.md`. If it is missing, run the `vault-setup` skill first. Never fake a write.
2. If `mode: git`, run `git -C <vault> pull --rebase --quiet`. If it fails, stop and report the error.

## Pick the path

- Folder: use one from `folders` in the config that fits (Meetings, People, Projects, ...). If none fits, use `Inbox/`. Do not invent new top-level folders without asking.
- Filename: `<Title>.md`. Strip these characters from the title: `: \ / # ^ [ ] |`. Keep spaces.
- If the user gives a name, use it exactly as the filename. Do not rename it or add a date.
- Only when the user gives no name and the note is a meeting or dated item, prefix the date: `2026-09-04 Standup.md`.
- If the file already exists, stop and use the `append-note` skill instead. Never overwrite.
- Run `mkdir -p "<vault>/<folder>"` before writing. Git does not keep empty folders.

## Write the note

```markdown
---
title: <Title>
date: <YYYY-MM-DD in the config timezone>
tags: [<one or two lowercase tags>]
source: <optional: url, "chat", or "email">
---

<Body. Short paragraphs. Bullet lists for lists.>

## Related
- [[<Existing note name>]]
```

- Wikilinks: `[[Note Name]]` with the exact filename without `.md`. Only link to notes that exist. Check with `ls` or `grep -rl` before linking. Do not link to notes you are guessing exist.
- Wikilinks inside YAML must be quoted: `related: "[[Note Name]]"`.
- No `# <Title>` heading at the top. Obsidian shows the filename as the title. Start with the body.
- No HTML. No `<br>`. Plain markdown only.
- Do not paste the whole chat. Write what the user would want to read next month.

## Commit and push (git mode only)

```bash
git -C <vault> add "<path>"
git -C <vault> commit -m "note: add <Title>"
git -C <vault> push
```

If push is rejected: `git -C <vault> pull --rebase` once, then push again. If it still fails, stop and show the error. Do not force push.

## Tell the user

One line: the path you wrote and the commit message. Example: `Wrote Meetings/2026-09-04 Standup.md (note: add 2026-09-04 Standup).`

## Rules

- Never delete or rename files.
- Never touch `.obsidian/`, `.git/`, or binary files.
- One commit per request.
- Ask before writing anything that looks like a secret, a password, or a card number.
