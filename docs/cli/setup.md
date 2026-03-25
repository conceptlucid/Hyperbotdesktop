---
summary: "CLI reference for `hyperbot setup` (initialize config + workspace)"
read_when:
  - You’re doing first-run setup without full CLI onboarding
  - You want to set the default workspace path
title: "setup"
---

# `hyperbot setup`

Initialize `~/.hyperbot/hyperbot.json` and the agent workspace.

Related:

- Getting started: [Getting started](/start/getting-started)
- CLI onboarding: [Onboarding (CLI)](/start/wizard)

## Examples

```bash
hyperbot setup
hyperbot setup --workspace ~/.hyperbot/workspace
```

To run onboarding via setup:

```bash
hyperbot setup --wizard
```
