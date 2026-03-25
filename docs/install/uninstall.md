---
summary: "Uninstall Ancient Claw completely (CLI, service, state, workspace)"
read_when:
  - You want to remove Ancient Claw from a machine
  - The gateway service is still running after uninstall
title: "Uninstall"
---

# Uninstall

Two paths:

- **Easy path** if `ancient-claw` is still installed.
- **Manual service removal** if the CLI is gone but the service is still running.

## Easy path (CLI still installed)

Recommended: use the built-in uninstaller:

```bash
ancient-claw uninstall
```

Non-interactive (automation / npx):

```bash
ancient-claw uninstall --all --yes --non-interactive
npx -y ancient-claw uninstall --all --yes --non-interactive
```

Manual steps (same result):

1. Stop the gateway service:

```bash
ancient-claw gateway stop
```

2. Uninstall the gateway service (launchd/systemd/schtasks):

```bash
ancient-claw gateway uninstall
```

3. Delete state + config:

```bash
rm -rf "${OPENCLAW_STATE_DIR:-$HOME/.ancient-claw}"
```

If you set `OPENCLAW_CONFIG_PATH` to a custom location outside the state dir, delete that file too.

4. Delete your workspace (optional, removes agent files):

```bash
rm -rf ~/.ancient-claw/workspace
```

5. Remove the CLI install (pick the one you used):

```bash
npm rm -g ancient-claw
pnpm remove -g ancient-claw
bun remove -g ancient-claw
```

6. If you installed the macOS app:

```bash
rm -rf /Applications/Ancient Claw.app
```

Notes:

- If you used profiles (`--profile` / `OPENCLAW_PROFILE`), repeat step 3 for each state dir (defaults are `~/.ancient-claw-<profile>`).
- In remote mode, the state dir lives on the **gateway host**, so run steps 1-4 there too.

## Manual service removal (CLI not installed)

Use this if the gateway service keeps running but `ancient-claw` is missing.

### macOS (launchd)

Default label is `ai.ancient-claw.gateway` (or `ai.ancient-claw.<profile>`; legacy `com.ancient-claw.*` may still exist):

```bash
launchctl bootout gui/$UID/ai.ancient-claw.gateway
rm -f ~/Library/LaunchAgents/ai.ancient-claw.gateway.plist
```

If you used a profile, replace the label and plist name with `ai.ancient-claw.<profile>`. Remove any legacy `com.ancient-claw.*` plists if present.

### Linux (systemd user unit)

Default unit name is `ancient-claw-gateway.service` (or `ancient-claw-gateway-<profile>.service`):

```bash
systemctl --user disable --now ancient-claw-gateway.service
rm -f ~/.config/systemd/user/ancient-claw-gateway.service
systemctl --user daemon-reload
```

### Windows (Scheduled Task)

Default task name is `Ancient Claw Gateway` (or `Ancient Claw Gateway (<profile>)`).
The task script lives under your state dir.

```powershell
schtasks /Delete /F /TN "Ancient Claw Gateway"
Remove-Item -Force "$env:USERPROFILE\.ancient-claw\gateway.cmd"
```

If you used a profile, delete the matching task name and `~\.ancient-claw-<profile>\gateway.cmd`.

## Normal install vs source checkout

### Normal install (install.sh / npm / pnpm / bun)

If you used `https://ancient-claw.ai/install.sh` or `install.ps1`, the CLI was installed with `npm install -g ancient-claw@latest`.
Remove it with `npm rm -g ancient-claw` (or `pnpm remove -g` / `bun remove -g` if you installed that way).

### Source checkout (git clone)

If you run from a repo checkout (`git clone` + `ancient-claw ...` / `bun run ancient-claw ...`):

1. Uninstall the gateway service **before** deleting the repo (use the easy path above or manual service removal).
2. Delete the repo directory.
3. Remove state + workspace as shown above.
