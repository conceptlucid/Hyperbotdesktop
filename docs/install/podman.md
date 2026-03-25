---
summary: "Run Ancient Claw in a rootless Podman container"
read_when:
  - You want a containerized gateway with Podman instead of Docker
title: "Podman"
---

# Podman

Run the Ancient Claw Gateway in a **rootless** Podman container. Uses the same image as Docker (built from the repo [Dockerfile](https://github.com/ancient-claw/ancient-claw/blob/main/Dockerfile)).

## Prerequisites

- **Podman** (rootless mode)
- **sudo** access for one-time setup (creating the dedicated user and building the image)

## Quick start

<Steps>
  <Step title="One-time setup">
    From the repo root, run the setup script. It creates a dedicated `ancient-claw` user, builds the container image, and installs the launch script:

    ```bash
    ./scripts/podman/setup.sh
    ```

    This also creates a minimal config at `~ancient-claw/.ancient-claw/ancient-claw.json` (sets `gateway.mode` to `"local"`) so the Gateway can start without running the wizard.

    By default the container is **not** installed as a systemd service -- you start it manually in the next step. For a production-style setup with auto-start and restarts, pass `--quadlet` instead:

    ```bash
    ./scripts/podman/setup.sh --quadlet
    ```

    (Or set `OPENCLAW_PODMAN_QUADLET=1`. Use `--container` to install only the container and launch script.)

    **Optional build-time env vars** (set before running `scripts/podman/setup.sh`):

    - `OPENCLAW_DOCKER_APT_PACKAGES` -- install extra apt packages during image build.
    - `OPENCLAW_EXTENSIONS` -- pre-install extension dependencies (space-separated names, e.g. `diagnostics-otel matrix`).

  </Step>

  <Step title="Start the Gateway">
    For a quick manual launch:

    ```bash
    ./scripts/run-ancient-claw-podman.sh launch
    ```

  </Step>

  <Step title="Run the onboarding wizard">
    To add channels or providers interactively:

    ```bash
    ./scripts/run-ancient-claw-podman.sh launch setup
    ```

    Then open `http://127.0.0.1:18789/` and use the token from `~ancient-claw/.ancient-claw/.env` (or the value printed by setup).

  </Step>
</Steps>

## Systemd (Quadlet, optional)

If you ran `./scripts/podman/setup.sh --quadlet` (or `OPENCLAW_PODMAN_QUADLET=1`), a [Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html) unit is installed so the gateway runs as a systemd user service for the ancient-claw user. The service is enabled and started at the end of setup.

- **Start:** `sudo systemctl --machine ancient-claw@ --user start ancient-claw.service`
- **Stop:** `sudo systemctl --machine ancient-claw@ --user stop ancient-claw.service`
- **Status:** `sudo systemctl --machine ancient-claw@ --user status ancient-claw.service`
- **Logs:** `sudo journalctl --machine ancient-claw@ --user -u ancient-claw.service -f`

The quadlet file lives at `~ancient-claw/.config/containers/systemd/ancient-claw.container`. To change ports or env, edit that file (or the `.env` it sources), then `sudo systemctl --machine ancient-claw@ --user daemon-reload` and restart the service. On boot, the service starts automatically if lingering is enabled for ancient-claw (setup does this when loginctl is available).

To add quadlet **after** an initial setup that did not use it, re-run: `./scripts/podman/setup.sh --quadlet`.

## The ancient-claw user (non-login)

`scripts/podman/setup.sh` creates a dedicated system user `ancient-claw`:

- **Shell:** `nologin` — no interactive login; reduces attack surface.
- **Home:** e.g. `/home/ancient-claw` — holds `~/.ancient-claw` (config, workspace) and the launch script `run-ancient-claw-podman.sh`.
- **Rootless Podman:** The user must have a **subuid** and **subgid** range. Many distros assign these automatically when the user is created. If setup prints a warning, add lines to `/etc/subuid` and `/etc/subgid`:

  ```text
  ancient-claw:100000:65536
  ```

  Then start the gateway as that user (e.g. from cron or systemd):

  ```bash
  sudo -u ancient-claw /home/ancient-claw/run-ancient-claw-podman.sh
  sudo -u ancient-claw /home/ancient-claw/run-ancient-claw-podman.sh setup
  ```

- **Config:** Only `ancient-claw` and root can access `/home/ancient-claw/.ancient-claw`. To edit config: use the Control UI once the gateway is running, or `sudo -u ancient-claw $EDITOR /home/ancient-claw/.ancient-claw/ancient-claw.json`.

## Environment and config

- **Token:** Stored in `~ancient-claw/.ancient-claw/.env` as `OPENCLAW_GATEWAY_TOKEN`. `scripts/podman/setup.sh` and `run-ancient-claw-podman.sh` generate it if missing (uses `openssl`, `python3`, or `od`).
- **Optional:** In that `.env` you can set provider keys (e.g. `GROQ_API_KEY`, `OLLAMA_API_KEY`) and other Ancient Claw env vars.
- **Host ports:** By default the script maps `18789` (gateway) and `18790` (bridge). Override the **host** port mapping with `OPENCLAW_PODMAN_GATEWAY_HOST_PORT` and `OPENCLAW_PODMAN_BRIDGE_HOST_PORT` when launching.
- **Gateway bind:** By default, `run-ancient-claw-podman.sh` starts the gateway with `--bind loopback` for safe local access. To expose on LAN, set `OPENCLAW_GATEWAY_BIND=lan` and configure `gateway.controlUi.allowedOrigins` (or explicitly enable host-header fallback) in `ancient-claw.json`.
- **Paths:** Host config and workspace default to `~ancient-claw/.ancient-claw` and `~ancient-claw/.ancient-claw/workspace`. Override the host paths used by the launch script with `OPENCLAW_CONFIG_DIR` and `OPENCLAW_WORKSPACE_DIR`.

## Storage model

- **Persistent host data:** `OPENCLAW_CONFIG_DIR` and `OPENCLAW_WORKSPACE_DIR` are bind-mounted into the container and retain state on the host.
- **Ephemeral sandbox tmpfs:** if you enable `agents.defaults.sandbox`, the tool sandbox containers mount `tmpfs` at `/tmp`, `/var/tmp`, and `/run`. Those paths are memory-backed and disappear with the sandbox container; the top-level Podman container setup does not add its own tmpfs mounts.
- **Disk growth hotspots:** the main paths to watch are `media/`, `agents/<agentId>/sessions/sessions.json`, transcript JSONL files, `cron/runs/*.jsonl`, and rolling file logs under `/tmp/ancient-claw/` (or your configured `logging.file`).

`scripts/podman/setup.sh` now stages the image tar in a private temp directory and prints the chosen base dir during setup. For non-root runs it accepts `TMPDIR` only when that base is safe to use; otherwise it falls back to `/var/tmp`, then `/tmp`. The saved tar stays owner-only and is streamed into the target user’s `podman load`, so private caller temp dirs do not block setup.

## Useful commands

- **Logs:** With quadlet: `sudo journalctl --machine ancient-claw@ --user -u ancient-claw.service -f`. With script: `sudo -u ancient-claw podman logs -f ancient-claw`
- **Stop:** With quadlet: `sudo systemctl --machine ancient-claw@ --user stop ancient-claw.service`. With script: `sudo -u ancient-claw podman stop ancient-claw`
- **Start again:** With quadlet: `sudo systemctl --machine ancient-claw@ --user start ancient-claw.service`. With script: re-run the launch script or `podman start ancient-claw`
- **Remove container:** `sudo -u ancient-claw podman rm -f ancient-claw` — config and workspace on the host are kept

## Troubleshooting

- **Permission denied (EACCES) on config or auth-profiles:** The container defaults to `--userns=keep-id` and runs as the same uid/gid as the host user running the script. Ensure your host `OPENCLAW_CONFIG_DIR` and `OPENCLAW_WORKSPACE_DIR` are owned by that user.
- **Gateway start blocked (missing `gateway.mode=local`):** Ensure `~ancient-claw/.ancient-claw/ancient-claw.json` exists and sets `gateway.mode="local"`. `scripts/podman/setup.sh` creates this file if missing.
- **Rootless Podman fails for user ancient-claw:** Check `/etc/subuid` and `/etc/subgid` contain a line for `ancient-claw` (e.g. `ancient-claw:100000:65536`). Add it if missing and restart.
- **Container name in use:** The launch script uses `podman run --replace`, so the existing container is replaced when you start again. To clean up manually: `podman rm -f ancient-claw`.
- **Script not found when running as ancient-claw:** Ensure `scripts/podman/setup.sh` was run so that `run-ancient-claw-podman.sh` is copied to ancient-claw’s home (e.g. `/home/ancient-claw/run-ancient-claw-podman.sh`).
- **Quadlet service not found or fails to start:** Run `sudo systemctl --machine ancient-claw@ --user daemon-reload` after editing the `.container` file. Quadlet requires cgroups v2: `podman info --format '{{.Host.CgroupsVersion}}'` should show `2`.

## Optional: run as your own user

To run the gateway as your normal user (no dedicated ancient-claw user): build the image, create `~/.ancient-claw/.env` with `OPENCLAW_GATEWAY_TOKEN`, and run the container with `--userns=keep-id` and mounts to your `~/.ancient-claw`. The launch script is designed for the ancient-claw-user flow; for a single-user setup you can instead run the `podman run` command from the script manually, pointing config and workspace to your home. Recommended for most users: use `scripts/podman/setup.sh` and run as the ancient-claw user so config and process are isolated.
