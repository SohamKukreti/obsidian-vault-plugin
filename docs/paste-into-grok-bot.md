# Paste these into your Grok Bot, one block at a time

For each block: copy everything from BEGIN to END, paste into the Bot chat, and send it as one message.

################ BEGIN vault-setup ################

Save everything below this line as a skill called "vault-setup". Keep the text exactly as written. Do not run it now, just save it.

---
name: vault-setup
description: One-time setup for the Obsidian vault skills. Asks for the vault's GitHub repo, branch, daily-note folder, and timezone, clones the vault, and saves the config. Use when the user says "set up my vault", when another vault skill finds no config, or when the user wants to change vault settings.
---

# Vault setup

Run this once per computer. Every other vault skill reads the config this writes.

## Where things live

Pick the base folder by environment:

- **Grok Bot** (the folder `/workspace` exists): base is `/workspace/obsidian-vault`.
- **Cursor or any laptop**: base is `~/.obsidian-vault-bot`.

Inside the base:

```
config.md   # settings, plain YAML frontmatter
vault/      # git clone of the vault (Grok Bot), or a symlink to the local vault (laptop)
```

If `config.md` already exists, show its values and ask what to change. Do not start over.

## Steps

1. **Check git.** Run `git --version`. If git is missing, stop and tell the user to install it. Do not try to write notes without git.

2. **Ask four things.** Ask them together in one message:
   - `repo`: GitHub `owner/repo` of the vault. It must be a private repo.
   - `branch`: default `main`.
   - `daily_folder` and `daily_format`: default `Daily` and `YYYY-MM-DD`.
   - `timezone`: an IANA name like `Asia/Kolkata` or `America/New_York`.

3. **Get the vault.**
   - Grok Bot: `git clone --branch <branch> https://github.com/<repo>.git <base>/vault`. If the clone asks for a login, stop and tell the user: "Open Agent Computer, take over, and sign in with `gh auth login` or store a fine-grained token with contents read and write on this repo only. Never paste the token in chat." Then retry once.
   - Laptop: ask for the local vault folder path. Symlink it to `<base>/vault`. Do not clone. Obsidian Git on the laptop handles sync.

4. **Check write access.** Run `git -C <base>/vault push --dry-run`. If it fails, report the exact error and stop. Do not create `config.md` until this passes.

5. **Learn the layout.** Run `ls <base>/vault` and note the top-level folders. Save them as `folders` in the config so the write skills can file notes where the user already keeps them.

6. **Write `config.md`:**

```markdown
---
repo: owner/repo
branch: main
vault: /workspace/obsidian-vault/vault
daily_folder: Daily
daily_format: YYYY-MM-DD
timezone: Asia/Kolkata
folders: [Daily, Meetings, People, Projects]
mode: git        # git on Grok Bot, local on a laptop
---
Created by vault-setup on 2026-09-04.
```

7. **Say what you did.** Show the config values and say "Vault ready. Try: add to today's daily note: setup done."

## Rules

- Never store a token, password, or cookie in `config.md` or anywhere else.
- Never touch `.obsidian/` or `.git/` inside the vault.
- If the repo is public, warn the user once. Notes in a public repo are public.

################ END vault-setup ################

################ BEGIN write-note ################

Save everything below this line as a skill called "write-note". Keep the text exactly as written. Do not run it now, just save it.

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

################ END write-note ################

################ BEGIN append-note ################

Save everything below this line as a skill called "append-note". Keep the text exactly as written. Do not run it now, just save it.

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

################ END append-note ################

################ BEGIN daily-note ################

Save everything below this line as a skill called "daily-note". Keep the text exactly as written. Do not run it now, just save it.

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

################ END daily-note ################

################ BEGIN search-vault ################

Save everything below this line as a skill called "search-vault". Keep the text exactly as written. Do not run it now, just save it.

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

################ END search-vault ################

