---
name: search-vault
description: Search the user's Obsidian vault by filename or text and return matching note paths with a short snippet each. Use when the user asks "what do I have on", "find my note about", "search my vault", or before linking to or appending to a note. Read only.
---

# Search the vault

Read only. This skill never writes, commits, or pushes.

## Before you search

1. Read the config. Grok Bot: `/workspace/obsidian-vault/config.md`. Laptop: `~/.obsidian-vault-bot/config.md`. If missing, run `vault-setup` first.
2. If `mode: git`, run `git -C <vault> pull --rebase --quiet` so results are current. If pull fails, search anyway and say the copy may be stale.

## How to search

Run both, in this order, and merge:

1. **Filenames:** `find <vault> -iname "*<term>*.md" -not -path "*/.obsidian/*" -not -path "*/.git/*"`
2. **Text:** `grep -ril --include="*.md" --exclude-dir=.obsidian --exclude-dir=.git "<term>" <vault>`

For several words, search each and rank notes that match more words higher. For a tag, search `#tag` and `tags:` lines.

## What to return

- At most 10 results. Say "more exist" if there are more.
- For each: the path relative to the vault, and one line of matching text (use `grep -i -m1 "<term>" <file>`). Trim to about 120 characters.
- Sort: filename matches first, then most recently modified (`ls -t`).
- If nothing matches, say so and suggest two other terms.

## Do not

- Do not print whole notes unless the user asks for one specific note.
- Do not list the entire vault.
- Do not search inside `.obsidian/`, `.git/`, or binary files.
