---
summary: "CLI reference for `hyperbot reset` (reset local state/config)"
read_when:
  - You want to wipe local state while keeping the CLI installed
  - You want a dry-run of what would be removed
title: "reset"
---

# `hyperbot reset`

Reset local config/state (keeps the CLI installed).

```bash
hyperbot backup create
hyperbot reset
hyperbot reset --dry-run
hyperbot reset --scope config+creds+sessions --yes --non-interactive
```

Run `hyperbot backup create` first if you want a restorable snapshot before removing local state.
