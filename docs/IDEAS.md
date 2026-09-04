# Plugin ideas in the Obsidian line. Researched 2026-09-04.

The shape: skills only, no server. Data lives as plain text files in a private git
repo. The Bot writes with git on its own computer. An app the user already uses
renders the files. Approval is "append only" or "open a PR".

## What already exists (so we skip it)

- Notes apps on the Cursor marketplace: Craft, Mem, Notion, Readwise, Supermemory. Obsidian and Logseq are not there.
- Plain text accounting: beancount.io ships a hosted MCP with OAuth for Cursor. No Grok Bot skill pack, no Gmail intake, no zero hosting path.
- Warranty tracker: a paste prompt on grokbots.page. Not a plugin.
- Finance plugins: Mercury, Brex, Ramp, Xero. All business. Nothing personal.
- Learning: Hugging Face, Exa. No flashcards. No reading to cards.
- Missing categories per awesome-grok-connectors: notes, health, home, travel, learning, family.

## Ranked ideas

### 1. Plain Text Money (my pick)
- What: a Bot reads receipts and bank alerts from the Gmail plugin and appends
  beancount or ledger entries to `main.beancount` in a private git repo. Fava on
  your laptop shows the dashboards.
- Skills: setup, add-transaction, import-from-email (routine, daily), monthly-close, ask-my-ledger.
- Why it fits: same git flow as Obsidian. Beancount runs on the Bot computer, so it can validate the file before commit.
- Why it can spread: the plain text accounting crowd is loud, technical, and already on GitHub. "My inbox books itself" is a post.
- Risk: money data. Private repo only. Bot never pays or moves money. Append only.

### 2. Cooklang Recipe Box (most creative)
- What: send the Bot a link, a photo, or "what nani made". It writes a `.cook`
  file (the cooklang plain text recipe format) to a git repo. The Cooklang app
  renders it and makes the shopping list.
- Skills: save-recipe, scale-recipe, weekly-plan, shopping-list (routine, Sunday).
- Why it fits: text files, git, existing app. Nothing like it exists for any agent.
- Why it can spread: food is universal. Screenshots look good. Cooklang has a small but active community with nothing AI native.
- Risk: photo to recipe needs the Bot to read images. Works, but check quality.

### 3. Anki Deck Maker
- What: the Bot turns Readwise highlights, articles, or your own notes into
  flashcards. It builds an `.apkg` with genanki on its computer and commits it.
  You double click to import.
- Skills: cards-from-text, cards-from-highlights (routine, weekly), review-deck.
- Why it fits: learning is an empty category. No Grok Bot or Cursor plugin does this.
- Catch: Anki import is a manual double click. AnkiConnect is localhost, so the Bot cannot reach it. Say this in the README.

### 4. Build in Public Blog (Hugo or Astro)
- What: the Bot drafts posts as markdown with `draft: true` into your static site
  repo and opens a PR. You merge. Netlify or Vercel deploys. Merge is the approval.
- Skills: draft-post, weekly-recap (routine, from git log and calendar), fix-frontmatter.
- Why it fits: same git flow, PR as the safety gate. Vercel and Netlify plugins already exist as the hands.
- Risk: "content bots" exist as prompts. This one wins on the PR gate and the site rebuild, not on writing.

### 5. todo.txt Inbox
- What: the Bot pulls tasks out of Gmail and Slack and appends them to
  `todo.txt` in git. Apps like Sleek and SimpleTask sync it.
- Skills: capture-task, morning-list (routine), done-list.
- Why it fits: one file, append only, tiny skills.
- Risk: Todoist plugin exists. This is for the plain text niche only. Smaller reach.

## Extensions of the Obsidian plugin (cheap wins later)

- Logseq mode: same skills, `journals/YYYY_MM_DD.md` and `pages/` layout.
- People notes: "log that I met X" writes to `People/X.md`. A personal CRM in the vault.
- Reading notes: Readwise plugin as intake, vault as output.

## Pattern to reuse in every one

1. Setup skill asks for repo, branch, folder layout, timezone. Saves to `/workspace/<plugin>/config.md`.
2. Every write: pull with rebase, read, append, validate, commit, push.
3. Never delete. Never touch app config folders. Ask before replacing.
4. One routine that runs daily and ends in a draft or an append, never a send.

## Sources

- Cursor marketplace: https://cursor.com/marketplace
- awesome-grok-connectors: https://github.com/rdmgator12/awesome-grok-connectors
- awesome-grokbot prompts: https://github.com/mergisi/awesome-grokbot
- Grok Bot use cases: https://docs.x.ai/grok-bot/use-cases
- Beancount MCP: https://beancount.io/blog/2026/06/30/beancount-mcp
- Plain text accounting: https://plaintextaccounting.org/
- Warranty prompt: https://www.grokbots.page/
- Discord plugin request: https://forum.cursor.com/t/official-discord-plugin-for-grok-bot-inbound-channels/168430
