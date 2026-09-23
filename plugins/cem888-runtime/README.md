# cem888-runtime — CEM888 lifecycle for Claude Cowork

Makes CEM888 native in Claude Cowork (and Claude Code). The model no longer has to remember to use CEM: host lifecycle hooks do it.

| Hook | CEM888 call | Effect |
|---|---|---|
| SessionStart | `cem_start_conversation` | Profile, current state and open work injected at session start |
| UserPromptSubmit | `cem_begin_external_turn` | Compiled current context injected before Claude reads your message (inhale) |
| PreToolUse | local owner-prohibition gate | Hard-NO enforced before any tool runs; fails closed for mutating tools |
| Stop | `cem_finish_external_turn` | Finished turn committed back to CEM888 exactly once (exhale) |

## Setup

1. Give the hooks a CEM888 Continuity credential, one of:
   - environment variable `CEM888_CONTINUITY_TOKEN`;
   - a file at `~/.cem888/continuity_token` containing only the token;
   - if the host runs hooks in a sandbox that cannot see your home folder: `secrets/continuity_token` inside the installed plugin folder (git-ignored; never commit it).
   The token is sent as `Authorization: Bearer <token>` to the Continuity MCP endpoint.
2. Optional: `CEM888_CONTINUITY_URL` (defaults to the CEM888 Continuity worker).
3. Install the plugin. Disable the manual `cem888-agent-continuity` skill while this plugin is active, or turns will be recorded twice.
4. Requires `python3` on the machine that runs the hooks.

## Owner prohibitions (hard NO)

Rules live in `policy/prohibitions.json` (shipped) and `~/.cem888/prohibitions.json` (merged on top). Matching is typed and deterministic:

```json
{ "id": "no-old-site", "reason": "Never deploy through the retired site repo.",
  "match": { "tool": "Bash", "command_contains": ["CEM888.AI-Site"] } }
```

Match keys (all present keys must match): `tool` (regex on tool name), `command_contains`, `path_prefix`, `url_host`, `args_contain`.

The gate also blocks the model from editing the policy files, the token, the plugin itself, and `~/.claude/settings*`.

## Guarantees and limits (honest classification)

- Inhale and exhale are automatic, not model-voluntary. They fail soft: a CEM outage never breaks the session.
- The prohibition gate covers every tool call that passes through the host's PreToolUse hook. Matching is literal/typed; semantic equivalents that avoid the listed strings are not caught (advisory layer via the skill only).
- Logs: `~/.cem888/cowork/hooks.log`.
- Status: BETA. Not yet certified on the frozen customer artifact. Hooks in Cowork plugins have a reported Windows issue; test on each OS before claiming support.
