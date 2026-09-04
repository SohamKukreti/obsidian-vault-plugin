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
   - `timezone`: ask "Which timezone are you in?" Accept anything: a city ("Delhi"), a short name ("IST", "PST"), or an offset ("GMT+5:30"). You turn it into the IANA name yourself (Delhi, IST, GMT+5:30 are all `Asia/Kolkata`). Confirm in one line: "IST, so Asia/Kolkata." Never ask the user for the IANA name.

3. **Get the vault.**
   - Laptop: ask for the local vault folder path. Symlink it to `<base>/vault`. Do not clone. Obsidian Git on the laptop handles sync. Skip the token step.
   - Grok Bot: git needs a way to log in to GitHub. First tell the user how to make a token:
     "Make a token: GitHub, Settings, Developer settings, Personal access tokens, Fine-grained tokens, Generate new token. Repository access: only your vault repo. Permissions: Contents, Read and write. Copy it."
     Then offer three ways to hand it over, in this order:

   **Way 1, paste it on the Bot's computer (token never enters the chat).** Write this script to `<base>/add-token.sh`:
   ```sh
   #!/bin/sh
   printf 'GitHub username: '; read u
   printf 'Paste token (hidden, then Enter): '; stty -echo; read t; stty echo; echo
   git config --global credential.helper store
   printf 'protocol=https\nhost=github.com\nusername=%s\npassword=%s\n' "$u" "$t" | git credential approve
   echo "Token stored. Go back to the chat and say: continue"
   ```
   Then tell the user: "Open Agent Computer, take over, open a terminal, run `sh <base>/add-token.sh`, paste the token, press Enter. Then say continue." When they say continue, delete the script and clone:
   ```bash
   rm <base>/add-token.sh
   git clone --branch <branch> https://github.com/<repo>.git <base>/vault
   ```

   **Way 2, send it in chat (fastest).** If the user sends the token in chat, store it and clone:
   ```bash
   git config --global credential.helper store
   printf 'protocol=https\nhost=github.com\nusername=<github username>\npassword=<token>\n' | git credential approve
   git clone --branch <branch> https://github.com/<repo>.git <base>/vault
   ```
   Never print the token back. Never write it to `config.md` or any note.

   **Way 3, browser login.** "Open Agent Computer, take over, run `gh auth login`, pick GitHub.com, HTTPS, browser. Then say continue." Then run `gh auth setup-git` and clone as above.

   If the clone fails with an auth error, show the error and ask which way to retry. Do not loop.

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
timezone_answer: IST
timezone: Asia/Kolkata
folders: [Daily, Meetings, People, Projects]
mode: git        # git on Grok Bot, local on a laptop
---
Created by vault-setup on 2026-09-04.
```

7. **Say what you did.** Show the config values and say "Vault ready. Try: add to today's daily note: setup done."

## Rules

- Never store a token, password, or cookie in `config.md` or any note. The git credential store on this computer is the only place a token may live.
- Never touch `.obsidian/` or `.git/` inside the vault.
- If the repo is public, warn the user once. Notes in a public repo are public.
