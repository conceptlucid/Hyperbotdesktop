---
summary: "CLI reference for `ancient-claw health` (gateway health endpoint via RPC)"
read_when:
  - You want to quickly check the running Gateway’s health
title: "health"
---

# `ancient-claw health`

Fetch health from the running Gateway.

```bash
ancient-claw health
ancient-claw health --json
ancient-claw health --verbose
```

Notes:

- `--verbose` runs live probes and prints per-account timings when multiple accounts are configured.
- Output includes per-agent session stores when multiple agents are configured.
