# pi-dev-config

Configuration repo for my pi.dev agent setup. Pi is a really great and lean agent to use with frontier AI and local LLMs alike. This repo is my customised setup which I have configured for my own needs in Linux. 

The SYSTEM.md and AGENTS.md are more like a restricted Claude while also staying as lean as I can make them for local AI. This is of course going to be a work in progress and anyone is free to take my setup.


## 1. Install Pi

Install Pi from the official website first:

https://pi.dev

Follow the official platform-specific instructions for Linux, macOS, or Windows.

## 2. Clone This Config Repo

### Windows PowerShell

Install [Git for Windows](https://git-scm.com/download/win), then open PowerShell:

```powershell
$Repo = "$env:USERPROFILE\Projects\pi-dev-config"
New-Item -ItemType Directory -Force "$env:USERPROFILE\Projects" | Out-Null
git clone https://github.com/suraj-lab/pi-dev-config.git $Repo
```

If you prefer SSH, use this clone URL instead:

```powershell
git clone git@github.com:suraj-lab/pi-dev-config.git $Repo
```

### Linux/macOS/WSL

```bash
mkdir -p ~/Projects
git clone https://github.com/suraj-lab/pi-dev-config.git ~/Projects/pi-dev-config
```

## 3. Sync Config Into Pi

Pi reads its config from `.pi` in your user home directory:

- Windows: `%USERPROFILE%\.pi`
- Linux/macOS/WSL: `~/.pi`

### Windows PowerShell

Use `robocopy` to copy the tracked config into your Windows Pi config directory while leaving secrets and runtime state alone:

```powershell
$Repo = "$env:USERPROFILE\Projects\pi-dev-config"
$PiDir = "$env:USERPROFILE\.pi"

New-Item -ItemType Directory -Force $PiDir | Out-Null

robocopy $Repo $PiDir /E `
  /XD ".git" "agent\sessions" "agent\bin" "context-mode" "logs" "cache" "sessions" `
  /XF "*auth*.json" "*token*" "*secret*" "*.key" "*.pem" "*.db" "*.sqlite" "*.sqlite3" "*.log" "*.backup" "*.bak" "*.tmp" "Thumbs.db" ".DS_Store"

# Robocopy uses exit codes 0-7 for success/copy-with-differences.
if ($LASTEXITCODE -le 7) { $global:LASTEXITCODE = 0 }
```

Restart Pi after syncing so it reloads the updated config.

### Linux/macOS/WSL

```bash
mkdir -p ~/.pi
rsync -av --exclude-from ~/Projects/pi-dev-config/.gitignore ~/Projects/pi-dev-config/ ~/.pi/
```

## 4. Add Local Secrets

Create or restore local-only auth/secrets separately.

Windows PowerShell:

```powershell
notepad "$env:USERPROFILE\.pi\agent\auth.json"
```

Linux/macOS/WSL:

```bash
$EDITOR ~/.pi/agent/auth.json
```

Never commit `agent/auth.json`. It is intentionally excluded from the sync commands and `.gitignore`.

## Updating an Existing Windows Setup

To pull the latest repo changes and sync them into Pi on Windows:

```powershell
$Repo = "$env:USERPROFILE\Projects\pi-dev-config"
$PiDir = "$env:USERPROFILE\.pi"

cd $Repo
git pull

robocopy $Repo $PiDir /E `
  /XD ".git" "agent\sessions" "agent\bin" "context-mode" "logs" "cache" "sessions" `
  /XF "*auth*.json" "*token*" "*secret*" "*.key" "*.pem" "*.db" "*.sqlite" "*.sqlite3" "*.log" "*.backup" "*.bak" "*.tmp" "Thumbs.db" ".DS_Store"
if ($LASTEXITCODE -le 7) { $global:LASTEXITCODE = 0 }
```

## Saving Local Pi Config Changes Back to the Repo

If you edit config directly under `.pi`, copy those safe files back into the repo and review the diff before committing.

Windows PowerShell:

```powershell
$Repo = "$env:USERPROFILE\Projects\pi-dev-config"
$PiDir = "$env:USERPROFILE\.pi"

robocopy $PiDir $Repo /E `
  /XD ".git" "agent\sessions" "agent\bin" "context-mode" "logs" "cache" "sessions" `
  /XF "*auth*.json" "*token*" "*secret*" "*.key" "*.pem" "*.db" "*.sqlite" "*.sqlite3" "*.log" "*.backup" "*.bak" "*.tmp" "Thumbs.db" ".DS_Store"
if ($LASTEXITCODE -le 7) { $global:LASTEXITCODE = 0 }

cd $Repo
git status
git diff
```

Linux/macOS/WSL:

```bash
cd ~/Projects/pi-dev-config
rsync -av --exclude-from .gitignore ~/.pi/ ./
git status
git diff
```

## Restoring Config From Repo on Linux/macOS/WSL

```bash
cd ~/Projects/pi-dev-config
rsync -av --exclude-from .gitignore ./ ~/.pi/
```

## Contents

- `agent/AGENTS.md` — project/system behavior rules
- `agent/SYSTEM.md` — core system instructions
- `agent/prompts/` — lean reusable prompts
- `agent/skills/` — local skills and references
- `agent/themes/` — Pi UI themes
- `agent/*.json` — safe agent config only
- `extensions/` — local extensions
- `web-search.json` — web search config

## Not Included

Secrets, sessions, caches, binaries, databases, logs, and generated runtime state are ignored.
