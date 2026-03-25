---
summary: "Uninstall HyperBot completely (CLI, service, state, workspace)"
read_when:
  - You want to remove HyperBot from a machine
  - The gateway service is still running after uninstall
title: "Uninstall"
---

# Uninstall

Two paths:

- **Easy path** if `hyperbot` is still installed.
- **Manual service removal** if the CLI is gone but the service is still running.

## Easy path (CLI still installed)

Recommended: use the built-in uninstaller:

```bash
hyperbot uninstall
```

Non-interactive (automation / npx):

```bash
hyperbot uninstall --all --yes --non-interactive
npx -y hyperbot uninstall --all --yes --non-interactive
```

Manual steps (same result):

1. Stop the gateway service:

```bash
hyperbot gateway stop
```

2. Uninstall the gateway service (launchd/systemd/schtasks):

```bash
hyperbot gateway uninstall
```

3. Delete state + config:

```bash
rm -rf "${OPENCLAW_STATE_DIR:-$HOME/.hyperbot}"
```

If you set `OPENCLAW_CONFIG_PATH` to a custom location outside the state dir, delete that file too.

4. Delete your workspace (optional, removes agent files):

```bash
rm -rf ~/.hyperbot/workspace
```

5. Remove the CLI install (pick the one you used):

```bash
npm rm -g hyperbot
pnpm remove -g hyperbot
bun remove -g hyperbot
```

6. If you installed the macOS app:

```bash
rm -rf /Applications/HyperBot.app
```

Notes:

- If you used profiles (`--profile` / `OPENCLAW_PROFILE`), repeat step 3 for each state dir (defaults are `~/.hyperbot-<profile>`).
- In remote mode, the state dir lives on the **gateway host**, so run steps 1-4 there too.

## Manual service removal (CLI not installed)

Use this if the gateway service keeps running but `hyperbot` is missing.

### macOS (launchd)

Default label is `ai.hyperbot.gateway` (or `ai.hyperbot.<profile>`; legacy `com.hyperbot.*` may still exist):

```bash
launchctl bootout gui/$UID/ai.hyperbot.gateway
rm -f ~/Library/LaunchAgents/ai.hyperbot.gateway.plist
```

If you used a profile, replace the label and plist name with `ai.hyperbot.<profile>`. Remove any legacy `com.hyperbot.*` plists if present.

### Linux (systemd user unit)

Default unit name is `hyperbot-gateway.service` (or `hyperbot-gateway-<profile>.service`):

```bash
systemctl --user disable --now hyperbot-gateway.service
rm -f ~/.config/systemd/user/hyperbot-gateway.service
systemctl --user daemon-reload
```

### Windows (Scheduled Task)

Default task name is `HyperBot Gateway` (or `HyperBot Gateway (<profile>)`).
The task script lives under your state dir.

```powershell
schtasks /Delete /F /TN "HyperBot Gateway"
Remove-Item -Force "$env:USERPROFILE\.hyperbot\gateway.cmd"
```

If you used a profile, delete the matching task name and `~\.hyperbot-<profile>\gateway.cmd`.

## Normal install vs source checkout

### Normal install (install.sh / npm / pnpm / bun)

If you used `https://hyperbot.ai/install.sh` or `install.ps1`, the CLI was installed with `npm install -g hyperbot@latest`.
Remove it with `npm rm -g hyperbot` (or `pnpm remove -g` / `bun remove -g` if you installed that way).

### Source checkout (git clone)

If you run from a repo checkout (`git clone` + `hyperbot ...` / `bun run hyperbot ...`):

1. Uninstall the gateway service **before** deleting the repo (use the easy path above or manual service removal).
2. Delete the repo directory.
3. Remove state + workspace as shown above.
