---
summary: "CLI reference for `hyperbot voicecall` (voice-call plugin command surface)"
read_when:
  - You use the voice-call plugin and want the CLI entry points
  - You want quick examples for `voicecall call|continue|status|tail|expose`
title: "voicecall"
---

# `hyperbot voicecall`

`voicecall` is a plugin-provided command. It only appears if the voice-call plugin is installed and enabled.

Primary doc:

- Voice-call plugin: [Voice Call](/plugins/voice-call)

## Common commands

```bash
hyperbot voicecall status --call-id <id>
hyperbot voicecall call --to "+15555550123" --message "Hello" --mode notify
hyperbot voicecall continue --call-id <id> --message "Any questions?"
hyperbot voicecall end --call-id <id>
```

## Exposing webhooks (Tailscale)

```bash
hyperbot voicecall expose --mode serve
hyperbot voicecall expose --mode funnel
hyperbot voicecall expose --mode off
```

Security note: only expose the webhook endpoint to networks you trust. Prefer Tailscale Serve over Funnel when possible.
