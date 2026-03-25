---
summary: "CLI reference for `ancient-claw config` (get/set/unset/file/validate)"
read_when:
  - You want to read or edit config non-interactively
title: "config"
---

# `ancient-claw config`

Config helpers for non-interactive edits in `ancient-claw.json`: get/set/unset/validate
values by path and print the active config file. Run without a subcommand to
open the configure wizard (same as `ancient-claw configure`).

## Examples

```bash
ancient-claw config file
ancient-claw config get browser.executablePath
ancient-claw config set browser.executablePath "/usr/bin/google-chrome"
ancient-claw config set agents.defaults.heartbeat.every "2h"
ancient-claw config set agents.list[0].tools.exec.node "node-id-or-name"
ancient-claw config set channels.discord.token --ref-provider default --ref-source env --ref-id DISCORD_BOT_TOKEN
ancient-claw config set secrets.providers.vaultfile --provider-source file --provider-path /etc/ancient-claw/secrets.json --provider-mode json
ancient-claw config unset plugins.entries.brave.config.webSearch.apiKey
ancient-claw config set channels.discord.token --ref-provider default --ref-source env --ref-id DISCORD_BOT_TOKEN --dry-run
ancient-claw config validate
ancient-claw config validate --json
```

## Paths

Paths use dot or bracket notation:

```bash
ancient-claw config get agents.defaults.workspace
ancient-claw config get agents.list[0].id
```

Use the agent list index to target a specific agent:

```bash
ancient-claw config get agents.list
ancient-claw config set agents.list[1].tools.exec.node "node-id-or-name"
```

## Values

Values are parsed as JSON5 when possible; otherwise they are treated as strings.
Use `--strict-json` to require JSON5 parsing. `--json` remains supported as a legacy alias.

```bash
ancient-claw config set agents.defaults.heartbeat.every "0m"
ancient-claw config set gateway.port 19001 --strict-json
ancient-claw config set channels.whatsapp.groups '["*"]' --strict-json
```

## `config set` modes

`ancient-claw config set` supports four assignment styles:

1. Value mode: `ancient-claw config set <path> <value>`
2. SecretRef builder mode:

```bash
ancient-claw config set channels.discord.token \
  --ref-provider default \
  --ref-source env \
  --ref-id DISCORD_BOT_TOKEN
```

3. Provider builder mode (`secrets.providers.<alias>` path only):

```bash
ancient-claw config set secrets.providers.vault \
  --provider-source exec \
  --provider-command /usr/local/bin/ancient-claw-vault \
  --provider-arg read \
  --provider-arg openai/api-key \
  --provider-timeout-ms 5000
```

4. Batch mode (`--batch-json` or `--batch-file`):

```bash
ancient-claw config set --batch-json '[
  {
    "path": "secrets.providers.default",
    "provider": { "source": "env" }
  },
  {
    "path": "channels.discord.token",
    "ref": { "source": "env", "provider": "default", "id": "DISCORD_BOT_TOKEN" }
  }
]'
```

```bash
ancient-claw config set --batch-file ./config-set.batch.json --dry-run
```

Batch parsing always uses the batch payload (`--batch-json`/`--batch-file`) as the source of truth.
`--strict-json` / `--json` do not change batch parsing behavior.

JSON path/value mode remains supported for both SecretRefs and providers:

```bash
ancient-claw config set channels.discord.token \
  '{"source":"env","provider":"default","id":"DISCORD_BOT_TOKEN"}' \
  --strict-json

ancient-claw config set secrets.providers.vaultfile \
  '{"source":"file","path":"/etc/ancient-claw/secrets.json","mode":"json"}' \
  --strict-json
```

## Provider Builder Flags

Provider builder targets must use `secrets.providers.<alias>` as the path.

Common flags:

- `--provider-source <env|file|exec>`
- `--provider-timeout-ms <ms>` (`file`, `exec`)

Env provider (`--provider-source env`):

- `--provider-allowlist <ENV_VAR>` (repeatable)

File provider (`--provider-source file`):

- `--provider-path <path>` (required)
- `--provider-mode <singleValue|json>`
- `--provider-max-bytes <bytes>`

Exec provider (`--provider-source exec`):

- `--provider-command <path>` (required)
- `--provider-arg <arg>` (repeatable)
- `--provider-no-output-timeout-ms <ms>`
- `--provider-max-output-bytes <bytes>`
- `--provider-json-only`
- `--provider-env <KEY=VALUE>` (repeatable)
- `--provider-pass-env <ENV_VAR>` (repeatable)
- `--provider-trusted-dir <path>` (repeatable)
- `--provider-allow-insecure-path`
- `--provider-allow-symlink-command`

Hardened exec provider example:

```bash
ancient-claw config set secrets.providers.vault \
  --provider-source exec \
  --provider-command /usr/local/bin/ancient-claw-vault \
  --provider-arg read \
  --provider-arg openai/api-key \
  --provider-json-only \
  --provider-pass-env VAULT_TOKEN \
  --provider-trusted-dir /usr/local/bin \
  --provider-timeout-ms 5000
```

## Dry run

Use `--dry-run` to validate changes without writing `ancient-claw.json`.

```bash
ancient-claw config set channels.discord.token \
  --ref-provider default \
  --ref-source env \
  --ref-id DISCORD_BOT_TOKEN \
  --dry-run

ancient-claw config set channels.discord.token \
  --ref-provider default \
  --ref-source env \
  --ref-id DISCORD_BOT_TOKEN \
  --dry-run \
  --json

ancient-claw config set channels.discord.token \
  --ref-provider vault \
  --ref-source exec \
  --ref-id discord/token \
  --dry-run \
  --allow-exec
```

Dry-run behavior:

- Builder mode: runs SecretRef resolvability checks for changed refs/providers.
- JSON mode (`--strict-json`, `--json`, or batch mode): runs schema validation plus SecretRef resolvability checks.
- Exec SecretRef checks are skipped by default during dry-run to avoid command side effects.
- Use `--allow-exec` with `--dry-run` to opt in to exec SecretRef checks (this may execute provider commands).
- `--allow-exec` is dry-run only and errors if used without `--dry-run`.

`--dry-run --json` prints a machine-readable report:

- `ok`: whether dry-run passed
- `operations`: number of assignments evaluated
- `checks`: whether schema/resolvability checks ran
- `checks.resolvabilityComplete`: whether resolvability checks ran to completion (false when exec refs are skipped)
- `refsChecked`: number of refs actually resolved during dry-run
- `skippedExecRefs`: number of exec refs skipped because `--allow-exec` was not set
- `errors`: structured schema/resolvability failures when `ok=false`

### JSON Output Shape

```json5
{
  ok: boolean,
  operations: number,
  configPath: string,
  inputModes: ["value" | "json" | "builder", ...],
  checks: {
    schema: boolean,
    resolvability: boolean,
    resolvabilityComplete: boolean,
  },
  refsChecked: number,
  skippedExecRefs: number,
  errors?: [
    {
      kind: "schema" | "resolvability",
      message: string,
      ref?: string, // present for resolvability errors
    },
  ],
}
```

Success example:

```json
{
  "ok": true,
  "operations": 1,
  "configPath": "~/.ancient-claw/ancient-claw.json",
  "inputModes": ["builder"],
  "checks": {
    "schema": false,
    "resolvability": true,
    "resolvabilityComplete": true
  },
  "refsChecked": 1,
  "skippedExecRefs": 0
}
```

Failure example:

```json
{
  "ok": false,
  "operations": 1,
  "configPath": "~/.ancient-claw/ancient-claw.json",
  "inputModes": ["builder"],
  "checks": {
    "schema": false,
    "resolvability": true,
    "resolvabilityComplete": true
  },
  "refsChecked": 1,
  "skippedExecRefs": 0,
  "errors": [
    {
      "kind": "resolvability",
      "message": "Error: Environment variable \"MISSING_TEST_SECRET\" is not set.",
      "ref": "env:default:MISSING_TEST_SECRET"
    }
  ]
}
```

If dry-run fails:

- `config schema validation failed`: your post-change config shape is invalid; fix path/value or provider/ref object shape.
- `SecretRef assignment(s) could not be resolved`: referenced provider/ref currently cannot resolve (missing env var, invalid file pointer, exec provider failure, or provider/source mismatch).
- `Dry run note: skipped <n> exec SecretRef resolvability check(s)`: dry-run skipped exec refs; rerun with `--allow-exec` if you need exec resolvability validation.
- For batch mode, fix failing entries and rerun `--dry-run` before writing.

## Subcommands

- `config file`: Print the active config file path (resolved from `OPENCLAW_CONFIG_PATH` or default location).

Restart the gateway after edits.

## Validate

Validate the current config against the active schema without starting the
gateway.

```bash
ancient-claw config validate
ancient-claw config validate --json
```
