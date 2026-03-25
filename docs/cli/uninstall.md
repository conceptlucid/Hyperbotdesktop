---
summary: "CLI reference for `ancient-claw uninstall` (remove gateway service + local data)"
read_when:
  - You want to remove the gateway service and/or local state
  - You want a dry-run first
title: "uninstall"
---

# `ancient-claw uninstall`

Uninstall the gateway service + local data (CLI remains).

```bash
ancient-claw backup create
ancient-claw uninstall
ancient-claw uninstall --all --yes
ancient-claw uninstall --dry-run
```

Run `ancient-claw backup create` first if you want a restorable snapshot before removing state or workspaces.
