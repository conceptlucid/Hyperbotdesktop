---
summary: "CLI reference for `hyperbot uninstall` (remove gateway service + local data)"
read_when:
  - You want to remove the gateway service and/or local state
  - You want a dry-run first
title: "uninstall"
---

# `hyperbot uninstall`

Uninstall the gateway service + local data (CLI remains).

```bash
hyperbot backup create
hyperbot uninstall
hyperbot uninstall --all --yes
hyperbot uninstall --dry-run
```

Run `hyperbot backup create` first if you want a restorable snapshot before removing state or workspaces.
