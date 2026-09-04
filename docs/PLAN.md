# Obsidian Vault plugin for Grok Bot. Corrected plan.

Date: 2026-09-04. Based on the official Grok Bot docs, the Cursor plugin docs and
template, the GitHub MCP server docs, and the Obsidian Git docs. Sources at the end.

## 1. What was wrong in the old plan

| Old plan said | What is true | Source |
|---|---|---|
| "There is no Obsidian plugin for Grok Bot." | Two exist. `obsidian-second-brain` (needs its own MCP server on cloud Grok Bot). `obsidian-bridge` (read only, self hosted). Nobody has a zero hosting, write capable one. That gap is still open. | awesome-grok-bot, both repos |
| Manifest is `plugin.json` with the agent-plugins.org schema. | Grok Bot plugins are Cursor plugins. Manifest is `.cursor-plugin/plugin.json`. Only `name` is required. The template also uses `displayName`, `version`, `description`, `author`, `logo`. | cursor.com/docs/plugins, cursor/plugin-template |
| Publish at cursor.com/marketplace/publish. | Correct. Every plugin is reviewed by hand. It must be open source. Each update is reviewed again. | cursor.com/docs/plugins |
| "Grok Bot check: install GitHub + this plugin." | Grok Bot has no side load for plugins. Plugins come only from the Cursor marketplace, shown in Settings, Plugins, Marketplace. Before listing, you test by pasting skill text into a Bot and asking it to save it as a skill. | docs.x.ai/grok-bot/teams-and-enterprises, skills page |
| GitHub connector uses `create_or_update_file` and `get_file_contents`. | Grok Bot has a native GitHub plugin. It covers repos, issues, pull requests, code search, Actions. Those tool names come from the GitHub MCP server. It is likely the same server, but I could not confirm the names inside Grok Bot. Do not build on them. | composio best-plugins, github-mcp-server |
| "Save as a bot memory." | Memory is automatic and split between per Bot and shared memory. There is no API to write a memory. Store setup facts in a file at `/workspace/obsidian-vault/config.md` instead. `/workspace` is the folder that survives updates. | aibuilderclub guide, docs |
| Google Drive or Dropbox as a second storage path. | Not confirmed as native connectors. Drop from v1. GitHub only. | docs, plugin lists |
| Obsidian Git "pull on start" and phone sync. | Obsidian Git has auto commit-and-sync on a timer and auto pull on startup. The exact setting labels are not in the docs I could reach. Mobile git is marked "very unstable". Phone needs Obsidian Sync or iCloud on top. | Obsidian Git README and docs |

## 2. The corrected design

### The one big change: use git, not the GitHub tool names

The Bot has its own cloud computer with files, a terminal, and command line
credentials. So the skill does not need any connector tool names. It clones the
vault once and uses plain git.

```
/workspace/obsidian-vault/
  config.md          # owner/repo, branch, daily folder, date format, timezone
  vault/             # git clone of the private vault repo
```

Write flow, every time:

1. `git -C /workspace/obsidian-vault/vault pull --rebase`
2. Read the note if it exists. Append. Never rewrite unless the user said "replace".
3. Write the file.
4. `git add`, `git commit -m "note: <what>"`, `git push`
5. If push is rejected: pull with rebase once, push again. If it fails again, stop and tell the user.

Why this is better than the connector path:
- No guessing of tool names.
- No blob SHA juggling. Git handles it.
- Search is `grep -ril` over the clone. Fast and free.
- Works the same in Cursor local test and in Grok Bot.

Auth, one time: the user opens Agent Computer, takes over, and runs
`gh auth login` or stores a fine grained PAT (contents read and write on the vault
repo only) in the git credential helper. The docs say secrets are typed by you on the
Bot's computer, never pasted in chat. Keep it that way.

### What v1 does

- New note. Path `Folder/Title.md`. Frontmatter `title`, `date`, `tags`. Body uses `[[wikilinks]]`.
- Append to a note. Default action when the note exists.
- Daily note. `Daily/YYYY-MM-DD.md` in the user's timezone. Create if missing.
- Search. Paths plus a 1 line snippet. Never dump the vault.
- Setup. Ask for repo, branch, daily folder, timezone. Save to `config.md`. Clone.

### Safety rules, in every write skill

- If `config.md` or the clone is missing, run setup. Never fake a write.
- Append by default. Ask before replacing. Never delete. Never rename trees.
- Never touch `.obsidian/`, `.git/`, or binary files.
- Filenames: no `: \ / # ^ [ ] |`.
- Wikilinks in YAML are quoted: `related: "[[Note]]"`.
- One commit per user request. Commit message starts with `note:`.

### Plugin layout

```
obsidian-vault/
  .cursor-plugin/plugin.json
  README.md
  LICENSE                       # MIT. Marketplace needs open source.
  assets/logo.svg
  skills/
    vault-setup/SKILL.md
    write-note/SKILL.md
    append-note/SKILL.md
    daily-note/SKILL.md
    search-vault/SKILL.md
```

`.cursor-plugin/plugin.json`:

```json
{
  "name": "obsidian-vault",
  "displayName": "Obsidian Vault",
  "version": "0.1.0",
  "description": "Write, append, search, and keep daily notes in an Obsidian vault stored in a private GitHub repo. No server. No hosting.",
  "author": { "name": "Soham Kukreti" },
  "logo": "assets/logo.svg"
}
```

Each `SKILL.md` needs frontmatter `name` (must match the folder name, kebab case)
and `description` (says what it does and when to use it). Keep each skill under
two screens.

## 3. Build, test, ship

### Prep the vault, once

1. Put the vault in a **private** GitHub repo. Add `.obsidian/workspace*.json` to `.gitignore`.
2. In Obsidian desktop, install Obsidian Git. Turn on auto pull on startup and the
   commit-and-sync timer (5 to 10 minutes is fine).
3. Phone: use Obsidian Sync or iCloud from the desktop. Do not run git on the phone.

### Build

1. Clone `cursor/plugin-template`. Copy `plugins/starter-simple` to `obsidian-vault`.
2. Write the manifest and the five skills.
3. Run `node scripts/validate-template.mjs` from the template. Fix every error.

### Test in Cursor (local)

1. Copy or symlink the folder to `~/.cursor/plugins/local/obsidian-vault`. Restart Cursor.
2. Say "file this meeting in my vault". Check the commit on GitHub. Pull in Obsidian.

### Test in Grok Bot (before the marketplace)

1. Create a Bot. Name it "Notes". Job: "keep my Obsidian vault".
2. Paste the `vault-setup` skill text. Say "save this as a skill called vault-setup".
   Repeat for the other four.
3. Run setup. Take over Agent Computer for the GitHub login.
4. Say "add to today's daily note: tested the plugin". Check GitHub, then Obsidian.
5. Run the same test twice in a row. The second run must append, not overwrite.

### Ship

1. Public GitHub repo for the plugin. Never the vault.
2. Submit at `cursor.com/marketplace/publish`. Wait for the manual review.
3. After listing, it shows in Grok Bot under Settings, Plugins, Marketplace.

## 4. Open checks. Do these first, they take 10 minutes

1. Is `git` on the Bot's computer? Ask a Bot to run `git --version`. If missing, add
   an install step to `vault-setup`.
2. Does the Bot keep a credential helper across sessions? Log in once, wait a day,
   push again.
3. Do Cursor marketplace skills show as `/skill-name` in the Grok Bot composer, or
   only as plugin context? Install any small marketplace plugin and look.

## 5. What this will not do

- Reach a vault that lives only on your laptop. Grok Bot cannot see it.
- Instant phone sync. Git pull is seconds to a minute on desktop. Phone needs Obsidian Sync.
- Merge two edits to the same line at the same time. Git will stop and ask.

## Sources

- Grok Bot skills and routines: https://docs.x.ai/grok-bot/skills-routines-and-automations
- Grok Bot computer and apps: https://docs.x.ai/grok-bot/computer-and-apps
- Grok Bot approvals and security: https://docs.x.ai/grok-bot/approvals-security-and-privacy
- Grok Bot teams (Cursor plugin policy): https://docs.x.ai/grok-bot/teams-and-enterprises
- Cursor plugins docs: https://cursor.com/docs/plugins
- Cursor skills docs: https://cursor.com/docs/context/skills
- Cursor plugin template: https://github.com/cursor/plugin-template
- GitHub MCP server tools: https://github.com/github/github-mcp-server
- Grok Bot native plugins list: https://composio.dev/content/best-grok-bot-plugins
- awesome-grok-bot catalog: https://github.com/ZeroPointRepo/awesome-grok-bot
- obsidian-second-brain: https://github.com/eugeniughelbur/obsidian-second-brain
- obsidian-bridge: https://glama.ai/mcp/servers/iamsupersocks/grokbot-obsidian-bridge
- Obsidian Git: https://github.com/Vinzent03/obsidian-git and https://publish.obsidian.md/git-doc
