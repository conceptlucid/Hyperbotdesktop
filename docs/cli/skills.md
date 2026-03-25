---
summary: "CLI reference for `ancient-claw skills` (search/install/update/list/info/check)"
read_when:
  - You want to see which skills are available and ready to run
  - You want to search, install, or update skills from ClawHub
  - You want to debug missing binaries/env/config for skills
title: "skills"
---

# `ancient-claw skills`

Inspect local skills and install/update skills from ClawHub.

Related:

- Skills system: [Skills](/tools/skills)
- Skills config: [Skills config](/tools/skills-config)
- ClawHub installs: [ClawHub](/tools/clawhub)

## Commands

```bash
ancient-claw skills search "calendar"
ancient-claw skills install <slug>
ancient-claw skills install <slug> --version <version>
ancient-claw skills update <slug>
ancient-claw skills update --all
ancient-claw skills list
ancient-claw skills list --eligible
ancient-claw skills info <name>
ancient-claw skills check
```

`search`/`install`/`update` use ClawHub directly and install into the active
workspace `skills/` directory. `list`/`info`/`check` still inspect the local
skills visible to the current workspace and config.
