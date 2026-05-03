## Overview

A digital garden is a public, curated selection of your notes — not a blog, not a journal, but an evolving collection of your thinking. This SOP walks through setting up Quartz on a Mac, connecting it to your Obsidian vault via iCloud, and hosting it for free on GitHub Pages.

**Mental model:** Your vault is your private thinking workspace. Your digital garden is the curated public output of it. Not everything in your vault goes public — only what you intentionally place in a `garden/` subfolder.

---

## Prerequisites

- Mac with terminal access
- Obsidian installed, vault syncing via iCloud
- GitHub account (repository must be public for free GitHub Pages)

---

## 1. Install Homebrew and Node.js

Homebrew is a package manager for Mac. Node.js (and npm) is required to run Quartz.

**Install Homebrew:**

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

After installation, Homebrew will print "Next steps" — run all three commands it gives you to add it to your PATH. They look like this:

```bash
echo >> /Users/YOUR_USERNAME/.zprofile
echo 'eval "$(/opt/homebrew/bin/brew shellenv zsh)"' >> /Users/YOUR_USERNAME/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv zsh)"
```

**Install Node.js:**

```bash
brew install node
```

**Verify:**

```bash
node --version
npm --version
```

> Homebrew installs to `/opt/homebrew/` — a system folder outside your home directory. You never need to navigate there manually. Your projects always live in `~/Developer/`.

---

## 2. Set Up Your Folder Structure

Create a dedicated developer folder (macOS treats `~/Developer` specially — Spotlight skips indexing it, which is important for performance with large projects):

```bash
mkdir -p ~/Developer/personal
```

Your home folder structure should look like this:

```
~/
  Developer/
    personal/
      quartz/        ← Quartz lives here
  Documents/         ← iCloud synced, for documents only
  Desktop/
  ...
```

> Never put git repos or code projects in `Documents/` — iCloud and git conflict with each other.

---

## 3. Clone and Install Quartz

```bash
cd ~/Developer/personal
git clone https://github.com/jackyzha0/quartz.git
cd quartz
npm i
```

**Initialize Quartz:**

```bash
npx quartz create
```

When prompted:

- Choose `Empty Quartz` to start fresh
- Select `Shortest Path` for link resolution

---

## 4. Connect Your Obsidian Vault via Symlink

Rather than copying notes manually, create a symlink so Quartz reads directly from your iCloud vault. Any note you place in the `garden/` subfolder of your vault will automatically appear in Quartz.

**First, find your vault name:**

```bash
ls ~/Library/Mobile\ Documents/com~apple~CloudDocs/Obsidian/
```

**Remove the default content folder and replace with a symlink:**

```bash
rm -rf ~/Developer/personal/quartz/content
ln -s ~/Library/Mobile\ Documents/com~apple~CloudDocs/Obsidian/YOUR-VAULT-NAME/garden ~/Developer/personal/quartz/content
```

Replace `YOUR-VAULT-NAME` with your actual vault folder name.

**Create the garden folder in your vault** (if it doesn't exist yet):

In Obsidian, create a folder called `garden/`. Any note you move here will be published. Notes outside this folder stay private.

> The symlink means there is no syncing step — `quartz/content/` and `iCloud/garden/` are literally the same folder accessed from two paths. Edits from your phone sync via iCloud and Quartz sees them instantly once iCloud finishes downloading.

---

## 5. Preview Locally

```bash
cd ~/Developer/personal/quartz
npx quartz build --serve
```

Open `http://localhost:8080` in your browser to preview your site.

---

## 6. Customize Your Site

Edit `quartz.config.ts` in the quartz root folder to set your site title and other settings:

```bash
nano quartz.config.ts
```

Find and update:

```typescript
pageTitle: "Your Name's Digital Garden",
```

**Front matter for notes** — add this YAML to the top of any note for extra control (not necessary):

```yaml
---
title: Note Title
draft: false
tags:
  - topic
---
```

Set `draft: true` on any note you want to temporarily hide from the published site.

---

## 7. Push to GitHub

**Create a new repository on GitHub** named `digital-garden` (must be public for free GitHub Pages).

**Connect your local repo:**

```bash
cd ~/Developer/personal/quartz
git remote set-url origin https://github.com/YOUR_USERNAME/digital-garden.git
```

**Authenticate with a Personal Access Token** (GitHub no longer accepts passwords):

1. Go to GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Click Generate new token (classic)
3. Name it (e.g. "macbook"), check both `repo` and `workflow` scopes
4. Copy the token immediately — you won't see it again

**Save the token in macOS Keychain** so you're not asked every time:

```bash
git config --global credential.helper osxkeychain
```

**Push:**

```bash
npx quartz sync
```

When prompted, enter your GitHub username and paste the token as the password. macOS Keychain saves it for future pushes.

> Quartz uses `v4` as its branch name, not `main`. Always run Quartz commands from the root quartz folder — the one containing `package.json`.

---

## 8. Enable GitHub Pages

1. Go to your repository on GitHub → **Settings → Pages**
2. Under Build and deployment, set source to **GitHub Actions**
3. GitHub will automatically run the deployment workflow included with Quartz
4. Your site will be live at `https://YOUR_USERNAME.github.io/digital-garden/`

---

## 9. Day-to-Day Publishing Workflow

1. Write notes anywhere in your Obsidian vault as usual
2. When a note is ready to publish, move it into the `garden/` subfolder
3. Run from the quartz folder:

```bash
npx quartz sync
```

That's it. The sync command builds the site and pushes to GitHub, which triggers the automatic deployment.

---

## Troubleshooting

|Error|Cause|Fix|
|---|---|---|
|`zsh: command not found: brew`|Homebrew not in PATH|Run the three PATH commands from the installer output|
|`zsh: command not found: npm`|Node.js not installed|`brew install node`|
|`Error: ENOENT: no such file or directory, open './package.json'`|Running command from wrong folder|`cd ~/Developer/personal/quartz` first|
|`src refspec main does not match any`|Wrong branch name|Use `git push -u origin v4` (Quartz uses v4, not main)|
|`refusing to allow a Personal Access Token to create workflow`|Token missing workflow scope|Edit token on GitHub, add `workflow` scope|
|`Invalid username or token`|Using GitHub password instead of token|Generate a Personal Access Token and use that|

---

## Storage Note

Keep video projects and large files (100 GB+) on an external SSD — not in your home folder or Documents. Internal Mac storage fills quickly and iCloud will try to sync large files. `~/Developer/` is for code only.