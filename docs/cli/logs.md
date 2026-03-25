---
summary: "CLI reference for `ancient-claw logs` (tail gateway logs via RPC)"
read_when:
  - You need to tail Gateway logs remotely (without SSH)
  - You want JSON log lines for tooling
title: "logs"
---

# `ancient-claw logs`

Tail Gateway file logs over RPC (works in remote mode).

Related:

- Logging overview: [Logging](/logging)

## Examples

```bash
ancient-claw logs
ancient-claw logs --follow
ancient-claw logs --json
ancient-claw logs --limit 500
ancient-claw logs --local-time
ancient-claw logs --follow --local-time
```

Use `--local-time` to render timestamps in your local timezone.
