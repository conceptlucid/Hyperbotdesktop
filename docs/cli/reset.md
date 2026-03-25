---
summary: "CLI reference for `ancient-claw reset` (reset local state/config)"
read_when:
  - You want to wipe local state while keeping the CLI installed
  - You want a dry-run of what would be removed
title: "reset"
---

# `ancient-claw reset`

Reset local config/state (keeps the CLI installed).

```bash
ancient-claw backup create
ancient-claw reset
ancient-claw reset --dry-run
ancient-claw reset --scope config+creds+sessions --yes --non-interactive
```

Run `ancient-claw backup create` first if you want a restorable snapshot before removing local state.
