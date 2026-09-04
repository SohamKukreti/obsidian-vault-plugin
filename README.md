# Obsidian Vault

A small plugin that lets Grok Bot (and Cursor) write into your Obsidian vault.

I wanted my Bot to file meeting notes and keep a daily log in Obsidian. The catch is
that a Bot lives on a cloud computer and can't see the folder on my laptop. The
existing integrations either need you to host a server or are read-only. So I did
the boring thing that works: the vault lives in a private GitHub repo, the Bot
clones it and writes plain markdown with git, and Obsidian Git on my laptop pulls
it back. From Obsidian's side it looks like the Bot wrote straight into the vault.

No server. No API keys. Five skill files.

## What it does

| You say | What happens |
|---|---|
| "set up my vault" | Asks four questions, clones the vault, saves a config. Once. |
| "file this as a note called Standup" | New note with frontmatter, tags, and links to notes that already exist. |
| "add to my Standup note: Priya joins Monday" | Appends one line. Touches nothing else. |
| "add to today's daily note: fixed the login bug" | Writes into `Daily/2026-09-04.md`, creating it if needed. |
| "what do I have on Raj" | Lists matching notes with a snippet. Read only. |

Every write is: pull, read, append, commit, push. One commit per request.

## Rules it follows

These are baked into the skill text, not optional.

- Append by default. Replacing a note needs an explicit yes, and it tells you what would be lost first.
- Never deletes, never renames, never force pushes.
- Never touches `.obsidian/`, `.git/`, or binary files.
- Never stores a token. You sign in on the Bot's computer yourself.
- Only links to notes that actually exist. No made-up wikilinks.

## Setting up your vault

You do this once.

1. Put your vault in a **private** GitHub repo. Add a `.gitignore`:
   ```
   .obsidian/workspace.json
   .obsidian/workspace-mobile.json
   .obsidian/plugins/*/data.json
   .trash/
   ```
   Then `git init -b main`, commit, add the remote, push.

2. In Obsidian on your desktop, install the community plugin **Git** (by Vinzent03).
   In its settings, under *Automatic*, set the commit-and-sync interval to 10 minutes
   and the pull interval to 5. Under *Pull*, turn on *Pull on startup*.

3. Phone: use Obsidian Sync or iCloud from the desktop. Don't run git on the phone.
   The Git plugin's own docs say mobile git is very unstable.

## Installing the plugin

**Grok Bot.** Settings → Plugins → Marketplace → Obsidian Vault → Add.
Until it's listed there, tell your Bot:

```
Clone https://github.com/SohamKukreti/obsidian-vault-plugin to /workspace/obsidian-vault-plugin.
For each file in skills/*/SKILL.md, save it as a skill using the name from its frontmatter.
Then list the skills you saved.
```

Or paste the blocks from `docs/paste-into-grok-bot.md` one at a time.

**Cursor.** Settings → Plugins → search "Obsidian Vault". Until it's listed, copy this
repo into `~/.cursor/plugins/local/obsidian-vault` (a real copy, not a symlink; Cursor
doesn't follow symlinks there) and reload the window.

## First run

Say `set up my vault`. It asks for:

- `repo`: `owner/name` of your vault repo
- `branch`: usually `main`
- daily folder and date format: defaults are `Daily` and `YYYY-MM-DD`
- timezone: an IANA name like `Asia/Kolkata`

On Grok Bot it clones the repo. When git asks for a login, open Agent Computer,
take over, and run `gh auth login` (or store a fine-grained token scoped to that one
repo). Don't paste a token into the chat.

On a laptop it just points at your local vault folder and lets Obsidian Git do the sync.

Then: `add to today's daily note: it works`. Pull in Obsidian. It should be there.

## Routines that work well

- Weekdays at 18:00: "Summarize what we did today into my daily note."
- After each meeting: "File the notes from this meeting."
- Sunday: "List the notes I touched this week."

## What it won't do

- Reach a vault that only lives on your laptop. The Bot can't see your machine.
- Instant phone sync. Git pull on desktop takes seconds to a minute.
- Resolve two people editing the same line at the same moment. Git stops and the Bot tells you.

## Layout

```
.cursor-plugin/plugin.json
assets/logo.svg
skills/
  vault-setup/SKILL.md
  write-note/SKILL.md
  append-note/SKILL.md
  daily-note/SKILL.md
  search-vault/SKILL.md
docs/
  paste-into-grok-bot.md   # all five skills, ready to paste
  PLAN.md                  # research notes and design decisions
  IDEAS.md                 # other plugins in the same shape
```

## Contributing

Skills are just markdown. If a skill does something dumb, edit the text and open a PR.
Run `node scripts/validate-template.mjs` from Cursor's
[plugin-template](https://github.com/cursor/plugin-template) to check frontmatter.

MIT.
