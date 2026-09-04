---
name: save-research
description: Turn research, findings, or a comparison the Bot just produced into a well-structured, linked note in the user's Obsidian vault, then commit and push. Use when the user says "save this research", "write these findings to Obsidian", "put this in my vault", or when a research task or routine finishes and the result should be kept. This is the main skill of the plugin.
---

# Save research to the vault

The Bot did the work in chat. This skill makes it a note the user can find in Obsidian next month.

## Before you write

1. Read the config. Grok Bot: `/workspace/obsidian-vault/config.md`. Laptop: `~/.obsidian-vault-bot/config.md`. If missing, run `vault-setup` first. Never fake a write.
2. If `mode: git`, run `git -C <vault> pull --rebase --quiet`. Stop on error.

## Pick the path

- Folder: `Research/` unless the config `folders` list has a better fit the user already uses (for example `Notes/Research` or a project folder). Never invent a new top-level folder without asking.
- Filename: a short topic title, three to six words, `<Title>.md`. Strip `: \ / # ^ [ ] |`. Example: `Research/Obsidian sync options for cloud agents.md`.
- If the user gave a name, use it exactly.
- If the file exists, do not overwrite. Append a dated `## Update <YYYY-MM-DD>` section at the end with the new findings, following the `append-note` rules.
- Run `mkdir -p "<vault>/<folder>"` first. Git does not keep empty folders.

## Write the note

```markdown
---
title: <Title>
date: <YYYY-MM-DD in the config timezone>
tags: [research, <one topic tag>]
status: draft
question: <the question the research answers, one line>
sources: <number of sources>
---

## TL;DR
- <three to five bullets. The answer, not the process.>

## Key findings
- <finding> [^1]
- <finding> [^2]

## Details
<Short sections with `###` headings. Only what the user would need to act. Numbers and quotes keep their source footnote.>

## Open questions
- <what is still unknown or unverified>

## Related
- [[<Existing note name>]]

## Sources
[^1]: <Title of source> - <url>
[^2]: <Title of source> - <url>
```

Rules for the content:

- Every number, date, price, or quote gets a footnote to its source. No source, say "unverified".
- Keep the user's own words for the question. Do not rewrite what they asked.
- Say what you are unsure about. A note that hides doubt is worse than no note.
- No `# <Title>` heading at the top. Obsidian shows the filename as the title.
- Wikilinks: `[[Note Name]]` only for notes that exist. Check with `grep -rl` or `find` before linking. Search the vault for the topic and link the two or three closest notes.
- Wikilinks inside YAML must be quoted.
- Plain markdown. No HTML.
- Do not paste the chat. Do not include the Bot's reasoning or tool calls.
- Aim for one screen. Long research gets a longer Details section, not a longer TL;DR.

## Also log it in today's daily note

After writing the research note, add one line to today's daily note using the `daily-note` rules:

```
- <HH:mm> Research: [[<Title>]] - <one line summary>
```

Create the daily note if it is missing. This is how the user finds what the Bot did today.

## Commit and push (git mode only)

One commit for both files:

```bash
git -C <vault> add "<research path>" "<daily path>"
git -C <vault> commit -m "research: <Title>"
git -C <vault> push
```

If push is rejected: `git -C <vault> pull --rebase` once, then push again. If it still fails, stop and show the error. Do not force push.

## Tell the user

Two lines. The research note path and the daily note line. Example:

```
Saved Research/Obsidian sync options for cloud agents.md (5 sources, 2 open questions).
Logged it in Daily/2026-09-04.md.
```

## Rules

- Never delete or rename files.
- Never touch `.obsidian/`, `.git/`, or binary files.
- Never write secrets, tokens, or private data from sources into the vault.
