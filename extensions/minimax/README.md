# MiniMax (HyperBot plugin)

Bundled MiniMax plugin for both:

- API-key provider setup (`minimax`)
- Coding Plan OAuth setup (`minimax-portal`)

## Enable

```bash
hyperbot plugins enable minimax
```

Restart the Gateway after enabling.

```bash
hyperbot gateway restart
```

## Authenticate

OAuth:

```bash
hyperbot models auth login --provider minimax-portal --set-default
```

API key:

```bash
hyperbot setup --wizard --auth-choice minimax-global-api
```

## Notes

- MiniMax OAuth uses a user-code login flow.
- OAuth currently targets the Coding Plan path.
