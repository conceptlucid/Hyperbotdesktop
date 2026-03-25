---
summary: "Run HyperBot in a rootless Podman container"
read_when:
  - You want a containerized gateway with Podman instead of Docker
title: "Podman"
---

# Podman

Run the HyperBot Gateway in a **rootless** Podman container. Uses the same image as Docker (built from the repo [Dockerfile](https://github.com/hyperbot/hyperbot/blob/main/Dockerfile)).

## Prerequisites

- **Podman** (rootless mode)
- **sudo** access for one-time setup (creating the dedicated user and building the image)

## Quick start

<Steps>
  <Step title="One-time setup">
    From the repo root, run the setup script. It creates a dedicated `hyperbot` user, builds the container image, and installs the launch script:

    ```bash
    ./scripts/podman/setup.sh
    ```

    This also creates a minimal config at `~hyperbot/.hyperbot/hyperbot.json` (sets `gateway.mode` to `"local"`) so the Gateway can start without running the wizard.

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
    ./scripts/run-hyperbot-podman.sh launch
    ```

  </Step>

  <Step title="Run the onboarding wizard">
    To add channels or providers interactively:

    ```bash
    ./scripts/run-hyperbot-podman.sh launch setup
    ```

    Then open `http://127.0.0.1:18789/` and use the token from `~hyperbot/.hyperbot/.env` (or the value printed by setup).

  </Step>
</Steps>

## Systemd (Quadlet, optional)

If you ran `./scripts/podman/setup.sh --quadlet` (or `OPENCLAW_PODMAN_QUADLET=1`), a [Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html) unit is installed so the gateway runs as a systemd user service for the hyperbot user. The service is enabled and started at the end of setup.

- **Start:** `sudo systemctl --machine hyperbot@ --user start hyperbot.service`
- **Stop:** `sudo systemctl --machine hyperbot@ --user stop hyperbot.service`
- **Status:** `sudo systemctl --machine hyperbot@ --user status hyperbot.service`
- **Logs:** `sudo journalctl --machine hyperbot@ --user -u hyperbot.service -f`

The quadlet file lives at `~hyperbot/.config/containers/systemd/hyperbot.container`. To change ports or env, edit that file (or the `.env` it sources), then `sudo systemctl --machine hyperbot@ --user daemon-reload` and restart the service. On boot, the service starts automatically if lingering is enabled for hyperbot (setup does this when loginctl is available).

To add quadlet **after** an initial setup that did not use it, re-run: `./scripts/podman/setup.sh --quadlet`.

## The hyperbot user (non-login)

`scripts/podman/setup.sh` creates a dedicated system user `hyperbot`:

- **Shell:** `nologin` — no interactive login; reduces attack surface.
- **Home:** e.g. `/home/hyperbot` — holds `~/.hyperbot` (config, workspace) and the launch script `run-hyperbot-podman.sh`.
- **Rootless Podman:** The user must have a **subuid** and **subgid** range. Many distros assign these automatically when the user is created. If setup prints a warning, add lines to `/etc/subuid` and `/etc/subgid`:

  ```text
  hyperbot:100000:65536
  ```

  Then start the gateway as that user (e.g. from cron or systemd):

  ```bash
  sudo -u hyperbot /home/hyperbot/run-hyperbot-podman.sh
  sudo -u hyperbot /home/hyperbot/run-hyperbot-podman.sh setup
  ```

- **Config:** Only `hyperbot` and root can access `/home/hyperbot/.hyperbot`. To edit config: use the Control UI once the gateway is running, or `sudo -u hyperbot $EDITOR /home/hyperbot/.hyperbot/hyperbot.json`.

## Environment and config

- **Token:** Stored in `~hyperbot/.hyperbot/.env` as `OPENCLAW_GATEWAY_TOKEN`. `scripts/podman/setup.sh` and `run-hyperbot-podman.sh` generate it if missing (uses `openssl`, `python3`, or `od`).
- **Optional:** In that `.env` you can set provider keys (e.g. `GROQ_API_KEY`, `OLLAMA_API_KEY`) and other HyperBot env vars.
- **Host ports:** By default the script maps `18789` (gateway) and `18790` (bridge). Override the **host** port mapping with `OPENCLAW_PODMAN_GATEWAY_HOST_PORT` and `OPENCLAW_PODMAN_BRIDGE_HOST_PORT` when launching.
- **Gateway bind:** By default, `run-hyperbot-podman.sh` starts the gateway with `--bind loopback` for safe local access. To expose on LAN, set `OPENCLAW_GATEWAY_BIND=lan` and configure `gateway.controlUi.allowedOrigins` (or explicitly enable host-header fallback) in `hyperbot.json`.
- **Paths:** Host config and workspace default to `~hyperbot/.hyperbot` and `~hyperbot/.hyperbot/workspace`. Override the host paths used by the launch script with `OPENCLAW_CONFIG_DIR` and `OPENCLAW_WORKSPACE_DIR`.

## Storage model

- **Persistent host data:** `OPENCLAW_CONFIG_DIR` and `OPENCLAW_WORKSPACE_DIR` are bind-mounted into the container and retain state on the host.
- **Ephemeral sandbox tmpfs:** if you enable `agents.defaults.sandbox`, the tool sandbox containers mount `tmpfs` at `/tmp`, `/var/tmp`, and `/run`. Those paths are memory-backed and disappear with the sandbox container; the top-level Podman container setup does not add its own tmpfs mounts.
- **Disk growth hotspots:** the main paths to watch are `media/`, `agents/<agentId>/sessions/sessions.json`, transcript JSONL files, `cron/runs/*.jsonl`, and rolling file logs under `/tmp/hyperbot/` (or your configured `logging.file`).

`scripts/podman/setup.sh` now stages the image tar in a private temp directory and prints the chosen base dir during setup. For non-root runs it accepts `TMPDIR` only when that base is safe to use; otherwise it falls back to `/var/tmp`, then `/tmp`. The saved tar stays owner-only and is streamed into the target user’s `podman load`, so private caller temp dirs do not block setup.

## Useful commands

- **Logs:** With quadlet: `sudo journalctl --machine hyperbot@ --user -u hyperbot.service -f`. With script: `sudo -u hyperbot podman logs -f hyperbot`
- **Stop:** With quadlet: `sudo systemctl --machine hyperbot@ --user stop hyperbot.service`. With script: `sudo -u hyperbot podman stop hyperbot`
- **Start again:** With quadlet: `sudo systemctl --machine hyperbot@ --user start hyperbot.service`. With script: re-run the launch script or `podman start hyperbot`
- **Remove container:** `sudo -u hyperbot podman rm -f hyperbot` — config and workspace on the host are kept

## Troubleshooting

- **Permission denied (EACCES) on config or auth-profiles:** The container defaults to `--userns=keep-id` and runs as the same uid/gid as the host user running the script. Ensure your host `OPENCLAW_CONFIG_DIR` and `OPENCLAW_WORKSPACE_DIR` are owned by that user.
- **Gateway start blocked (missing `gateway.mode=local`):** Ensure `~hyperbot/.hyperbot/hyperbot.json` exists and sets `gateway.mode="local"`. `scripts/podman/setup.sh` creates this file if missing.
- **Rootless Podman fails for user hyperbot:** Check `/etc/subuid` and `/etc/subgid` contain a line for `hyperbot` (e.g. `hyperbot:100000:65536`). Add it if missing and restart.
- **Container name in use:** The launch script uses `podman run --replace`, so the existing container is replaced when you start again. To clean up manually: `podman rm -f hyperbot`.
- **Script not found when running as hyperbot:** Ensure `scripts/podman/setup.sh` was run so that `run-hyperbot-podman.sh` is copied to hyperbot’s home (e.g. `/home/hyperbot/run-hyperbot-podman.sh`).
- **Quadlet service not found or fails to start:** Run `sudo systemctl --machine hyperbot@ --user daemon-reload` after editing the `.container` file. Quadlet requires cgroups v2: `podman info --format '{{.Host.CgroupsVersion}}'` should show `2`.

## Optional: run as your own user

To run the gateway as your normal user (no dedicated hyperbot user): build the image, create `~/.hyperbot/.env` with `OPENCLAW_GATEWAY_TOKEN`, and run the container with `--userns=keep-id` and mounts to your `~/.hyperbot`. The launch script is designed for the hyperbot-user flow; for a single-user setup you can instead run the `podman run` command from the script manually, pointing config and workspace to your home. Recommended for most users: use `scripts/podman/setup.sh` and run as the hyperbot user so config and process are isolated.
