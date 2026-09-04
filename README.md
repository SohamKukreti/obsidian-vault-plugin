# Obsidian Vault

Get Grok Bot's research into your Obsidian vault as real notes.

I use a Bot to research things: compare tools, dig through docs, watch a topic. The
results were stuck in chat. What I actually wanted was a note in my vault, with the
sources, that I can open on my laptop next month and link from other notes.

The catch is that a Bot lives on a cloud computer and can't see the folder on my
laptop. The existing Obsidian integrations either need you to host a server or are
read-only. So this does the boring thing that works: the vault lives in a private
GitHub repo, the Bot clones it and writes markdown with git, and Obsidian Git on my
laptop pulls it back. From Obsidian's side it looks like the Bot wrote straight into
the vault.

No server. No API keys. Six skill files.

## What it does

| You say | What happens |
|---|---|
| "save this research" | A structured note in `Research/`: TL;DR, key findings with footnoted sources, open questions, links to related notes. Plus one line in today's daily note so you can find it. |
| "add to today's daily note: ..." | Writes into `Daily/2026-09-04.md`, creating it if needed. |
| "file this as a note called X" | New note with frontmatter and tags. |
| "add to my X note: ..." | Appends one line. Touches nothing else. |
| "what do I have on Y" | Lists matching notes with a snippet. Read only. |
| "set up my vault" | Four questions, clones the vault, saves a config. Once. |

The research note looks like this in Obsidian:

```
Research/Obsidian sync options for cloud agents.md

  question: how can a cloud agent write to a local vault
  tags: research, obsidian     sources: 5     status: draft

  TL;DR
  - Local REST API only works on the same machine
  - Git-backed vault is the only zero-hosting write path
  - ...

  Key findings
  - obsidian-bridge is read-only by design [^2]
  ...
  Open questions
  - Does the Bot computer keep git credentials between sessions?

  Sources
  [^1] ...
```

Every write is: pull, read, append, commit, push. One commit per request.

## Rules it follows

Baked into the skill text, not optional.

- Every number or quote in a research note has a footnote to its source. No source, it says "unverified".
- Append by default. Replacing a note needs an explicit yes, and it tells you what would be lost first.
- Never deletes, never renames, never force pushes.
- Never touches `.obsidian/`, `.git/`, or binary files.
- Never stores a token. You sign in on the Bot's computer yourself.
- Only links to notes that actually exist. No made-up wikilinks.

## Setting up your vault

Once.

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

Then ask the Bot to research something, and say `save this research`. Pull in
Obsidian. It's there.

## Routines that work well

- Monday 09:00: "Research what changed in [topic] this week and save it to my vault."
- Weekdays at 18:00: "Summarize what we did today into my daily note."
- When a research task finishes: "Save the findings to Obsidian."

## What it won't do

- Reach a vault that only lives on your laptop. The Bot can't see your machine.
- Instant phone sync. Git pull on desktop takes seconds to a minute.
- Resolve two people editing the same line at the same moment. Git stops and the Bot tells you.

## Layout

```
.cursor-plugin/plugin.json
assets/logo.svg
skills/
  save-research/SKILL.md   # the main one
  daily-note/SKILL.md
  write-note/SKILL.md
  append-note/SKILL.md
  search-vault/SKILL.md
  vault-setup/SKILL.md
docs/
  paste-into-grok-bot.md   # all six skills, ready to paste
  PLAN.md                  # research notes and design decisions
  IDEAS.md                 # other plugins in the same shape
```

## Contributing

Skills are just markdown. If a skill does something dumb, edit the text and open a PR.
Run `node scripts/validate-template.mjs` from Cursor's
[plugin-template](https://github.com/cursor/plugin-template) to check frontmatter.

MIT.
