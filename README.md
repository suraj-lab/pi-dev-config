# pi-dev-config

Private configuration repo for Suraj's pi.dev agent setup.

## 1. Install Pi

Install Pi from the official website first:

https://pi.dev

Follow the official platform-specific instructions for Linux, macOS, or Windows.

## 2. Clone This Config Repo

```bash
mkdir -p ~/Projects
git clone git@github.com:<your-user>/pi-dev-config.git ~/Projects/pi-dev-config
```

## 3. Sync Config Into Pi

Linux/macOS:

```bash
mkdir -p ~/.pi
rsync -av --exclude-from ~/Projects/pi-dev-config/.gitignore ~/Projects/pi-dev-config/ ~/.pi/
```

Windows:

Use WSL if possible and follow the Linux commands above.

Otherwise, copy the repo contents into your Pi config directory manually, excluding:
- `agent/auth.json`
- sessions
- caches
- logs
- databases
- binaries

## 4. Add Local Secrets

Create or restore local-only auth/secrets separately:

```bash
$EDITOR ~/.pi/agent/auth.json
```

Never commit `agent/auth.json`.

## Updating Repo From Current System

```bash
cd ~/Projects/pi-dev-config
rsync -av --exclude-from .gitignore ~/.pi/ ./
git status
```

## Restoring Config From Repo

```bash
cd ~/Projects/pi-dev-config
rsync -av --exclude-from .gitignore ./ ~/.pi/
```

## Contents

- `agent/AGENTS.md` — project/system behavior rules
- `agent/SYSTEM.md` — core system instructions
- `agent/prompts/` — lean reusable prompts
- `agent/skills/` — local skills and references
- `agent/*.json` — safe agent config only
- `extensions/` — local extensions
- `web-search.json` — web search config

## Not Included

Secrets, sessions, caches, binaries, databases, logs, and generated runtime state are ignored.

Keep this repo private.
