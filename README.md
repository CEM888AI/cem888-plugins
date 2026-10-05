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

License: CEM888 Source-Available Commercial License — source-available, commercial, never converts to open source. **Free for personal, non-commercial use:** you may read, audit, copy, modify and distribute the plugins for yourself. **Nothing that makes money, or builds a business, is free** — business production use, earning from its output, resale, embedding, white-labeling, OEM use, and providing the plugins to third parties as a hosted or managed service all require a paid licence from CEM Unlimited LLC. Non-retroactive: plugins released under Business Source License 1.1 or the Elastic License 2.0 remain under those terms. Commercial licensing: creator@cem888.ai
