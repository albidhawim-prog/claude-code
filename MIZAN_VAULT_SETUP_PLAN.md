# Mizan Vault — Setup Plan (handoff to local Claude Code instance)

> This document was drafted by a cloud Claude Code session on `claude/create-mizan-repo-sOCTB`. The cloud session has restricted GitHub access and cannot create new repos or touch the user's machines, so a local Claude Code instance should pick up from here.

---

## 1. Context for the local instance

**The user is non-technical.** Talk to them in plain language, one step at a time, and verify each step before moving on. Do not dump multi-step instructions in a single message.

**The goal:** Build a private "project hub" for a project called **Mizan**. The hub must be:
- Accessible from all of the user's devices: Android phone, Windows desktop, Linux CLI, iPhone, iPad, MacBook, Windows laptop
- Editable by the user from any device
- Writable by Claude (you) — the user wants to chat with Claude on the go and have important takeaways automatically saved into the hub
- Private (the user explicitly asked about data privacy)

**Architecture chosen (after discussion with the user):**
- **Vault format:** Obsidian (the user already has Obsidian and prefers it for privacy — notes are plain markdown files on their own devices)
- **Sync backbone:** A **private GitHub repository** named `mizan-vault`, holding the markdown files
- **Per-device sync:** The Obsidian community plugin **Obsidian Git** (by Vinzent03) on each device, configured to auto-pull on startup and auto-push every few minutes
- **Claude bridge:** Local Claude Code instance, run from inside the cloned vault folder, can directly read/edit/commit/push markdown files. When the user says *"save this to Mizan"*, you write the file, commit, push — and within a few minutes it appears on every other device

**What was ruled out and why:**
- Notion: too cloud-dependent, user prefers local-first
- iCloud Drive sync: Apple-only, doesn't cover Android/Windows/Linux
- Google Drive / Dropbox / OneDrive: Obsidian on iOS doesn't sync these natively
- Obsidian Sync (paid): would solve sync but doesn't give Claude a way to write into the vault — defeats the "live bridge" requirement

---

## 2. Prerequisites — already in place

The user has already created both repos under the GitHub account `mizanpressco-hash`:

- **Vault repo:** https://github.com/mizanpressco-hash/mizan-vault — this is what this plan sets up.
- **Code repo:** https://github.com/mizanpressco-hash/mizan-project — for the actual project code; out of scope for this plan but linked from the vault later (see §7).

Before starting Phase A, confirm with the user:

1. They can sign in to GitHub on the MacBook as `mizanpressco-hash`.
2. They are sitting at the **MacBook**. Get the MacBook working end-to-end before touching any other device.
3. (Sanity check) Visit https://github.com/mizanpressco-hash/mizan-vault in a browser. It should load (it's private, so only the signed-in owner sees it). If it 404s, the user is signed in as the wrong account — fix that first.

### Device order

The MacBook is **not currently available**. Start on the **Windows workstation** instead — it's the most capable device on hand and the easiest to debug. The order is:

1. **Windows workstation** (Phase D below) — set up vault + verify push/pull
2. **Live bridge** (Phase B) — connect Claude Code to the vault on the workstation
3. **Android Fold 7** (Phase E)
4. **iPad** (Phase F)
5. **iPhone** (Phase F)
6. **MacBook**, **Windows laptop**, **Linux CLI** — whenever they come online (Phases A, D, C respectively)

Don't move past step 2 until you've confirmed Claude can write a note from the workstation and it appears on github.com.

---

## 3. Recommended vault folder structure

After cloning the empty repo, create this structure on the MacBook (the first device). The auto-push will replicate it to GitHub and from there to every other device.

```
mizan-vault/
├── README.md                          (already exists from repo init)
├── 00-Inbox/                          quick captures, sort later
│   └── .gitkeep
├── 01-Strategy/
│   ├── Frameworks/                    e.g. the "strategy_ops" framework the user mentioned
│   ├── Decisions/                     decision log entries
│   └── Vision/
├── 02-Operations/
│   ├── Tasks/
│   └── Processes/
├── 03-Knowledge/
│   ├── Research/
│   └── References/
├── 04-Chats-and-Captures/             where Claude writes chat takeaways
├── 05-Code-Links/                     notes about the code repo (for later)
└── _Templates/
    ├── Decision.md
    ├── Framework.md
    └── Chat-Capture.md
```

`.gitkeep` is an empty file used to make git track an otherwise-empty folder. Add one in any folder you want to commit while it has no content yet.

### Template contents

**`_Templates/Decision.md`**
```markdown
# Decision: <title>

- **Date:** YYYY-MM-DD
- **Status:** proposed | accepted | superseded
- **Owner:** <who decides>

## Context
<what's the situation>

## Options considered
1. ...
2. ...

## Decision
<what we picked>

## Why
<the reasoning>

## Consequences
<what this commits us to, what it rules out>
```

**`_Templates/Framework.md`**
```markdown
# Framework: <name>

- **Date captured:** YYYY-MM-DD
- **Source:** <chat / meeting / book / etc.>

## What it is
<one-paragraph summary>

## When to use it
<situations this applies to>

## The framework
<steps, principles, or diagram>

## Examples
<concrete uses>

## Related
- [[link to related framework or decision]]
```

**`_Templates/Chat-Capture.md`**
```markdown
# Chat Capture: <topic>

- **Date:** YYYY-MM-DD
- **Device:** <where the chat happened>
- **Tags:** #strategy #ideas

## Summary
<2–3 sentence summary>

## Key points
- ...
- ...

## Action items
- [ ] ...

## Raw
<paste of the relevant chat snippet>
```

---

## 4. Step-by-step setup (run with the user, one step at a time)

### Phase A — MacBook (first, get it working end-to-end)

1. **Install GitHub Desktop** from `desktop.github.com`. This is the simplest way to handle GitHub login on a Mac — it stores credentials system-wide so the Obsidian plugin can use them invisibly.
2. **Sign in** to GitHub Desktop with the user's account.
3. **Clone the repo via GitHub Desktop**: File → Clone repository → pick `mizanpressco-hash/mizan-vault` → choose `~/Documents/mizan-vault` as the local path → Clone.
4. **Install Obsidian** from `obsidian.md` if not already installed.
5. **Open the vault**: Obsidian → "Open folder as vault" → choose `~/Documents/mizan-vault`. Trust the author when prompted.
6. **Enable community plugins**: Settings (gear) → Community plugins → "Turn on community plugins."
7. **Install Obsidian Git**: Browse → search `Obsidian Git` (author: Vinzent03) → Install → Enable.
8. **Configure Obsidian Git** (Settings → Obsidian Git):
   - Vault backup interval (minutes): `5`
   - Auto pull on startup: **on**
   - Pull updates on startup: **on**
   - Auto push interval (minutes): `5`
9. **Create the folder structure** from §3. Drop the three template files in `_Templates/`.
10. **Verify push works**: Make a test note in `00-Inbox/test.md`. Wait 5 minutes (or in Obsidian, open Command Palette → "Obsidian Git: Push"). Refresh the GitHub repo page in a browser — `00-Inbox/test.md` should appear.
11. **Verify pull works**: On github.com, edit the test file in the browser and commit. Back in Obsidian, Command Palette → "Obsidian Git: Pull". The change should land locally.

**Do not move past Phase A until both push and pull verify cleanly.**

### Phase B — Connect Claude to the vault (the live bridge)

This is what makes "save this to Mizan" actually work. Run this on whichever device is hosting the Claude Code session — initially the **Windows workstation**.

**Windows (workstation):**
1. Open a terminal — PowerShell, Windows Terminal, or WSL all work. WSL is recommended if Claude Code is already installed there.
2. `cd` into the cloned vault folder, e.g. `cd C:\Users\<user>\Documents\mizan-vault` (or `cd /mnt/c/Users/<user>/Documents/mizan-vault` from WSL).
3. Start a Claude Code session: `claude`
4. The session now has direct file access to the vault. When the user says *"save this framework as 'X' in Mizan"*, Claude writes `01-Strategy/Frameworks/X.md`, commits, and pushes — and Obsidian Git on every other device pulls it in.
5. Test: ask Claude in that session to "create a framework note called 'hello world' using the framework template." Claude should write the file and run `git add . && git commit -m "..." && git push`. Confirm it appears on github.com.

**MacBook (when available):** Same flow from `~/Documents/mizan-vault` in a terminal.

### Phase C — Linux CLI machine

1. Install git and gh (`sudo apt install git gh` or distro equivalent).
2. `gh auth login` → follow prompts, pick HTTPS, authenticate via browser.
3. `git clone https://github.com/mizanpressco-hash/mizan-vault.git ~/mizan-vault`
4. (Optional) Install Obsidian via Flatpak/AppImage if the Linux box has a GUI. Otherwise the user can just edit markdown with their preferred editor.
5. To pull updates: `cd ~/mizan-vault && git pull`. To push: `git add . && git commit -m "..." && git push`. (No Obsidian Git plugin here — but they can run a local Claude Code session against this folder too.)

### Phase D — Windows desktop and Windows laptop

Same as MacBook:
1. Install GitHub Desktop, sign in as `mizanpressco-hash`, clone `mizanpressco-hash/mizan-vault` to e.g. `C:\Users\<user>\Documents\mizan-vault`.
2. Install Obsidian.
3. Open folder as vault.
4. Install + enable Obsidian Git plugin with the same settings as Phase A step 8.
5. Verify push and pull the same way.

### Phase E — Android

1. Install Obsidian from the Play Store.
2. Create a **fine-grained Personal Access Token** at github.com → Settings → Developer settings → Personal access tokens → Fine-grained tokens → Generate new token:
   - Resource owner: `mizanpressco-hash`
   - Repository access: **Only select repositories** → `mizan-vault`
   - Permissions: Contents = Read and write, Metadata = Read-only
   - Expiration: 90 days (set a calendar reminder to rotate)
   - Copy the token immediately — it's shown only once.
3. In Obsidian on Android: open Obsidian → Create new vault → name it `mizan-vault` → location: pick a folder (use Obsidian's app-internal storage, not /sdcard, to avoid Android storage permission issues).
4. Enable community plugins → install **Obsidian Git** → enable.
5. In Obsidian Git settings on Android:
   - Authentication → Username: `mizanpressco-hash`
   - Authentication → Password/Token: paste the fine-grained PAT
   - Then: Command Palette → "Obsidian Git: Clone an existing remote repo" → URL: `https://github.com/mizanpressco-hash/mizan-vault.git`
6. Same auto-pull/push settings as Phase A step 8.
7. Verify by making a small edit on Android and watching it appear on the MacBook (and vice versa).

### Phase F — iPhone and iPad

Two options. Try Option 1 first; fall back to Option 2 if the vault grows large or sync feels unreliable.

**Option 1 — Obsidian Git plugin natively (free):**
- Install Obsidian from the App Store.
- Same flow as Android: install Obsidian Git plugin, create the PAT (or reuse Android's), clone the repo from inside Obsidian.
- Known limitation: large vaults can be slow on iOS because the plugin uses a JavaScript git implementation. Fine for note-sized vaults.

**Option 2 — Working Copy + Obsidian (one-time ~$25 for full Working Copy features):**
- Install **Working Copy** from the App Store.
- In Working Copy: Repositories → + → Clone → paste the repo URL → authenticate with the PAT → clone.
- In Working Copy: tap the repo → Settings → "Setup Files.app integration" → enable.
- In Obsidian: Create new vault → "Open folder as vault" → navigate via the Files picker to the Working Copy repo → select.
- Pull/push happens in Working Copy (it can auto-fetch). Obsidian just edits the files.

---

## 5. Things to confirm with the user along the way

- **Don't put secrets in the vault.** Even though it's private, GitHub is not designed for credentials. No passwords, no API keys, no payment info. If they need that, recommend a password manager (1Password, Bitwarden) — separate concern.
- **PAT rotation reminder.** When they create the fine-grained token in Phase E, tell them to add a calendar reminder for the expiration date.
- **First merge conflict will surprise them.** If they edit the same note on two devices before either has synced, Obsidian Git will show a conflict. Walk them through it the first time it happens — don't pre-explain.
- **The user mentioned a "strategy_ops" framework** they wanted to share. Once Phase A is done, ask them to send it and save it as `01-Strategy/Frameworks/Strategy-Ops.md` using the framework template. That's a nice first real piece of content for the vault.

---

## 6. What "done" looks like

- The user can open Obsidian on any of their 7 devices and see the same Mizan vault.
- The user can chat with Claude (locally on the MacBook, run from inside the vault folder) and say *"save this to Mizan"* — and the note appears on the iPhone within ~5 minutes.
- A future, separate GitHub repo will hold the actual project code; the vault's `05-Code-Links/` folder will hold notes that reference it. That's a follow-up, not part of this plan.

---

## 7. Open items / future work

- **Code repo already exists:** https://github.com/mizanpressco-hash/mizan-project — the actual project code lives there. Once it has content, drop a note in `05-Code-Links/` of the vault that points at it (e.g. a markdown file listing the repo URL, key entry-point files, and how the code maps to the strategy notes). That's the simplest version of the "live layer between chats and code" the user asked for.
- Decide whether to install Claude Code on Android/iOS (the user mentioned wanting on-the-go access — currently Claude on mobile is via the chat apps, not Claude Code). For now, on-the-go capture works by: chat with Claude in the mobile app → ask it to format the takeaway as markdown → user pastes it into Obsidian on their phone → auto-pushes via Obsidian Git. Not as smooth as desktop but works.
- Revisit whether Obsidian Sync (paid) is worth adding *on top of* GitHub for faster mobile sync. Not needed initially.
