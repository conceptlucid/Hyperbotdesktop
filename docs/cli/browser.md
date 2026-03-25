---
summary: "CLI reference for `hyperbot browser` (profiles, tabs, actions, Chrome MCP, and CDP)"
read_when:
  - You use `hyperbot browser` and want examples for common tasks
  - You want to control a browser running on another machine via a node host
  - You want to attach to your local signed-in Chrome via Chrome MCP
title: "browser"
---

# `hyperbot browser`

Manage HyperBot’s browser control server and run browser actions (tabs, snapshots, screenshots, navigation, clicks, typing).

Related:

- Browser tool + API: [Browser tool](/tools/browser)

## Common flags

- `--url <gatewayWsUrl>`: Gateway WebSocket URL (defaults to config).
- `--token <token>`: Gateway token (if required).
- `--timeout <ms>`: request timeout (ms).
- `--browser-profile <name>`: choose a browser profile (default from config).
- `--json`: machine-readable output (where supported).

## Quick start (local)

```bash
hyperbot browser profiles
hyperbot browser --browser-profile hyperbot start
hyperbot browser --browser-profile hyperbot open https://example.com
hyperbot browser --browser-profile hyperbot snapshot
```

## Profiles

Profiles are named browser routing configs. In practice:

- `hyperbot`: launches or attaches to a dedicated HyperBot-managed Chrome instance (isolated user data dir).
- `user`: controls your existing signed-in Chrome session via Chrome DevTools MCP.
- custom CDP profiles: point at a local or remote CDP endpoint.

```bash
hyperbot browser profiles
hyperbot browser create-profile --name work --color "#FF5A36"
hyperbot browser create-profile --name chrome-live --driver existing-session
hyperbot browser delete-profile --name work
```

Use a specific profile:

```bash
hyperbot browser --browser-profile work tabs
```

## Tabs

```bash
hyperbot browser tabs
hyperbot browser open https://docs.hyperbot.ai
hyperbot browser focus <targetId>
hyperbot browser close <targetId>
```

## Snapshot / screenshot / actions

Snapshot:

```bash
hyperbot browser snapshot
```

Screenshot:

```bash
hyperbot browser screenshot
```

Navigate/click/type (ref-based UI automation):

```bash
hyperbot browser navigate https://example.com
hyperbot browser click <ref>
hyperbot browser type <ref> "hello"
```

## Existing Chrome via MCP

Use the built-in `user` profile, or create your own `existing-session` profile:

```bash
hyperbot browser --browser-profile user tabs
hyperbot browser create-profile --name chrome-live --driver existing-session
hyperbot browser create-profile --name brave-live --driver existing-session --user-data-dir "~/Library/Application Support/BraveSoftware/Brave-Browser"
hyperbot browser --browser-profile chrome-live tabs
```

This path is host-only. For Docker, headless servers, Browserless, or other remote setups, use a CDP profile instead.

## Remote browser control (node host proxy)

If the Gateway runs on a different machine than the browser, run a **node host** on the machine that has Chrome/Brave/Edge/Chromium. The Gateway will proxy browser actions to that node (no separate browser control server required).

Use `gateway.nodes.browser.mode` to control auto-routing and `gateway.nodes.browser.node` to pin a specific node if multiple are connected.

Security + remote setup: [Browser tool](/tools/browser), [Remote access](/gateway/remote), [Tailscale](/gateway/tailscale), [Security](/gateway/security)
