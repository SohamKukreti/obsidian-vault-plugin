---
name: append-note
description: Add text to the end of an existing note in the user's Obsidian vault without rewriting it, then commit and push. Use when the user says "add to", "append", "note that ... in", or names a note that already exists. Append is the default for existing notes; replacing needs explicit permission.
---

# Append to an existing note

## Before you write

1. Read the config. Grok Bot: `/workspace/obsidian-vault/config.md`. Laptop: `~/.obsidian-vault-bot/config.md`. If missing, run `vault-setup` first.
2. If `mode: git`, run `git -C <vault> pull --rebase --quiet`. Stop on error.

## Find the note

- Exact name first: `find <vault> -name "<Name>.md" -not -path "*/.obsidian/*"`.
- If not found, case-insensitive: `find <vault> -iname "*<name>*.md" -not -path "*/.obsidian/*"`.
- One match: use it. Several: list them and ask which. None: offer to create it with `write-note`.

## Read, then append

1. **Read the whole file first.** Never write to a note you have not read in this turn.
2. Keep the frontmatter and everything above untouched.
3. Append at the end. If the note has a section that clearly fits (for example `## Log` or `## Notes`), append inside that section instead. Say which you did.
4. Format of the appended block:

```markdown

- <YYYY-MM-DD HH:mm> <text>
```

For longer additions use a heading with the date:

```markdown

## <YYYY-MM-DD> <short heading>
<text>
```

5. Use the config timezone for the timestamp.
6. Do not rewrite, reorder, or "clean up" existing text. Do not fix typos elsewhere in the note.

## Replace only when told

Rewrite an existing note only if the user says "replace", "rewrite", or "overwrite" for that note. Before you do, show a one-line summary of what will be lost and ask "Replace it?". Wait for a yes.

## Commit and push (git mode only)

```bash
git -C <vault> add "<path>"
git -C <vault> commit -m "note: append to <Name>"
git -C <vault> push
```

If push is rejected: `git -C <vault> pull --rebase` once, then push again. If it still fails, stop and show the error. Do not force push.

## Tell the user

One line: the note path, where you appended (end or which section), and the commit message.

## Rules

- Never delete or rename files.
- Never touch `.obsidian/`, `.git/`, or binary files.
- One commit per request.
