---
summary: "Gateway runtime on macOS (external launchd service)"
read_when:
  - Packaging Ancient Claw.app
  - Debugging the macOS gateway launchd service
  - Installing the gateway CLI for macOS
title: "Gateway on macOS"
---

# Gateway on macOS (external launchd)

Ancient Claw.app no longer bundles Node/Bun or the Gateway runtime. The macOS app
expects an **external** `ancient-claw` CLI install, does not spawn the Gateway as a
child process, and manages a per‑user launchd service to keep the Gateway
running (or attaches to an existing local Gateway if one is already running).

## Install the CLI (required for local mode)

Node 24 is the default runtime on the Mac. Node 22 LTS, currently `22.14+`, still works for compatibility. Then install `ancient-claw` globally:

```bash
npm install -g ancient-claw@<version>
```

The macOS app’s **Install CLI** button runs the same flow via npm/pnpm (bun not recommended for Gateway runtime).

## Launchd (Gateway as LaunchAgent)

Label:

- `ai.ancient-claw.gateway` (or `ai.ancient-claw.<profile>`; legacy `com.ancient-claw.*` may remain)

Plist location (per‑user):

- `~/Library/LaunchAgents/ai.ancient-claw.gateway.plist`
  (or `~/Library/LaunchAgents/ai.ancient-claw.<profile>.plist`)

Manager:

- The macOS app owns LaunchAgent install/update in Local mode.
- The CLI can also install it: `ancient-claw gateway install`.

Behavior:

- “Ancient Claw Active” enables/disables the LaunchAgent.
- App quit does **not** stop the gateway (launchd keeps it alive).
- If a Gateway is already running on the configured port, the app attaches to
  it instead of starting a new one.

Logging:

- launchd stdout/err: `/tmp/ancient-claw/ancient-claw-gateway.log`

## Version compatibility

The macOS app checks the gateway version against its own version. If they’re
incompatible, update the global CLI to match the app version.

## Smoke check

```bash
ancient-claw --version

OPENCLAW_SKIP_CHANNELS=1 \
OPENCLAW_SKIP_CANVAS_HOST=1 \
ancient-claw gateway --port 18999 --bind loopback
```

Then:

```bash
ancient-claw gateway call health --url ws://127.0.0.1:18999 --timeout 3000
```
