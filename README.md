# CEM888 plugins

Official plugin marketplace for [CEM888](https://cem888.ai) — the reliability, continuity and control runtime underneath AI agents.

## Install in Claude Cowork (desktop)

1. Open the Claude desktop app → **Cowork** → **Customize** → **Plugins**.
2. Choose **Add marketplace** and enter: `https://github.com/CEM888AI/cem888-plugins`
3. Find **cem888-runtime** in the listing and click **Install**.
4. Put your CEM888 Continuity token in `~/.cem888/continuity_token` (your CEM888 installer does this for you).

## Install in Claude Code

```
/plugin marketplace add CEM888AI/cem888-plugins
/plugin install cem888-runtime@cem888
```

## What cem888-runtime does

| Host event | CEM888 action |
|---|---|
| Session start | Loads your agent's profile, current state and open work |
| Every message | Injects compiled authoritative context before Claude reads it (inhale) |
| Before every tool | Enforces your owner hard-NO rules; blocks before execution |
| After every response | Commits the finished turn back to CEM888 exactly once (exhale) |

Status: **BETA**. See `plugins/cem888-runtime/README.md` for setup, rules format, and the honest enforcement limits.

License: Elastic License 2.0 (ELv2) — source-available, never converts. Use, copy, distribute and modify freely; you may not provide the plugins to third parties as a hosted or managed service, and you may not move, change, disable or circumvent the license key functionality. Business production use, resale, embedding, white-labeling and OEM use require a commercial license from CEM Unlimited LLC. Plugins released on or after 2026-10-01 under Business Source License 1.1 remain under those terms. Commercial licensing: creator@cem888.ai
