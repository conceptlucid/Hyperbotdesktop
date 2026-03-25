---
summary: "CLI reference for `hyperbot devices` (device pairing + token rotation/revocation)"
read_when:
  - You are approving device pairing requests
  - You need to rotate or revoke device tokens
title: "devices"
---

# `hyperbot devices`

Manage device pairing requests and device-scoped tokens.

## Commands

### `hyperbot devices list`

List pending pairing requests and paired devices.

```
hyperbot devices list
hyperbot devices list --json
```

Pending request output includes the requested role and scopes so approvals can
be reviewed before you approve.

### `hyperbot devices remove <deviceId>`

Remove one paired device entry.

```
hyperbot devices remove <deviceId>
hyperbot devices remove <deviceId> --json
```

### `hyperbot devices clear --yes [--pending]`

Clear paired devices in bulk.

```
hyperbot devices clear --yes
hyperbot devices clear --yes --pending
hyperbot devices clear --yes --pending --json
```

### `hyperbot devices approve [requestId] [--latest]`

Approve a pending device pairing request. If `requestId` is omitted, HyperBot
automatically approves the most recent pending request.

Note: if a device retries pairing with changed auth details (role/scopes/public
key), HyperBot supersedes the previous pending entry and issues a new
`requestId`. Run `hyperbot devices list` right before approval to use the
current ID.

```
hyperbot devices approve
hyperbot devices approve <requestId>
hyperbot devices approve --latest
```

### `hyperbot devices reject <requestId>`

Reject a pending device pairing request.

```
hyperbot devices reject <requestId>
```

### `hyperbot devices rotate --device <id> --role <role> [--scope <scope...>]`

Rotate a device token for a specific role (optionally updating scopes).

```
hyperbot devices rotate --device <deviceId> --role operator --scope operator.read --scope operator.write
```

### `hyperbot devices revoke --device <id> --role <role>`

Revoke a device token for a specific role.

```
hyperbot devices revoke --device <deviceId> --role node
```

## Common options

- `--url <url>`: Gateway WebSocket URL (defaults to `gateway.remote.url` when configured).
- `--token <token>`: Gateway token (if required).
- `--password <password>`: Gateway password (password auth).
- `--timeout <ms>`: RPC timeout.
- `--json`: JSON output (recommended for scripting).

Note: when you set `--url`, the CLI does not fall back to config or environment credentials.
Pass `--token` or `--password` explicitly. Missing explicit credentials is an error.

## Notes

- Token rotation returns a new token (sensitive). Treat it like a secret.
- These commands require `operator.pairing` (or `operator.admin`) scope.
- `devices clear` is intentionally gated by `--yes`.
- If pairing scope is unavailable on local loopback (and no explicit `--url` is passed), list/approve can use a local pairing fallback.

## Token drift recovery checklist

Use this when Control UI or other clients keep failing with `AUTH_TOKEN_MISMATCH` or `AUTH_DEVICE_TOKEN_MISMATCH`.

1. Confirm current gateway token source:

```bash
hyperbot config get gateway.auth.token
```

2. List paired devices and identify the affected device id:

```bash
hyperbot devices list
```

3. Rotate operator token for the affected device:

```bash
hyperbot devices rotate --device <deviceId> --role operator
```

4. If rotation is not enough, remove stale pairing and approve again:

```bash
hyperbot devices remove <deviceId>
hyperbot devices list
hyperbot devices approve <requestId>
```

5. Retry client connection with the current shared token/password.

Related:

- [Dashboard auth troubleshooting](/web/dashboard#if-you-see-unauthorized-1008)
- [Gateway troubleshooting](/gateway/troubleshooting#dashboard-control-ui-connectivity)
