# cem888-runtime — CEM888 lifecycle for Claude Cowork

Makes CEM888 native in Claude Cowork (and Claude Code). The model no longer has to remember to use CEM: host lifecycle hooks do it.

| Hook | CEM888 call | Effect |
|---|---|---|
| SessionStart | `cem_start_conversation` | Profile, current state and open work injected at session start |
| UserPromptSubmit | `cem_begin_external_turn` | Compiled current context injected before Claude reads your message (inhale) |
| PreToolUse | local owner-prohibition gate | Hard-NO enforced before any tool runs; fails closed for mutating tools |
| PostToolUse | `cem_context_for` | Mid-work refresh: compiled context re-injected after the canvas changes (mutating calls always; reads sampled) |
| Stop | `cem_finish_external_turn` | Finished turn committed back to CEM888 exactly once (exhale), in the background |

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

## Mid-work refresh

Turn-granularity inhale happens on every user message. The `PostToolUse` hook adds the *other* injection point: it re-injects compiled context **while the agent works** — but only when state may actually have changed. Mutating calls always refresh; reads are sampled. A read does not dirty the canvas, so refreshing on every `grep` would burn context for no gain.

Resolved **defaults → `~/.cem888/posttool_refresh.json` → env** (env wins):

| Knob | Env | Default | Meaning |
|---|---|---|---|
| `enabled` | `CEM888_POSTTOOL_ENABLED` | `true` | master switch |
| `on_mutating` | `CEM888_POSTTOOL_ON_MUTATING` | `true` | refresh after every mutating call |
| `read_every` | `CEM888_POSTTOOL_READ_EVERY` | `8` | refresh every Nth READ call (`0` = never) |
| `limit` | `CEM888_POSTTOOL_LIMIT` | `4` | items the service may return |
| `max_chars` | `CEM888_POSTTOOL_MAX_CHARS` | `4000` | hard cap on injected size |
| `refresh_tool` | `CEM888_POSTTOOL_TOOL` | `cem_current_state` | which read tool compiles the refresh |

```json
{ "enabled": true, "on_mutating": true, "read_every": 8, "limit": 4, "max_chars": 4000, "refresh_tool": "cem_current_state" }
```

The configured tool is tried first, then `cem_current_state` → `cem_open_work` → `cem_recall` in order; the first that answers wins. A single broken service-side tool must not take the refresh down. `limit` is sent only to the payload tools (`cem_recall`, `cem_context_for`).

**Known service defect (as of 2026-09-24): `cem_context_for` fails on every argument combination** — including bare `task` with `limit=1` — returning `{"isError": true, "text": "Context bootstrap metadata exceeds output budget"}`. `cem_current_state`, `cem_open_work` and `cem_recall` all work. That is why the default is `cem_current_state`; switch back via `refresh_tool` once the defect is fixed.

`~/.cem888/posttool_refresh.json` is in the gate's self-protected set — the model cannot edit it. CEM's own read tools never trigger a refresh (no refresh-on-refresh), and a refresh resets the read counter because it is a synchronization point.

## Guarantees and limits (honest classification)

- Inhale and exhale are automatic, not model-voluntary. They fail soft: a CEM outage never breaks the session.
- Exhale runs detached: the Stop hook returns immediately and the commit finishes in the background. If CEM is unreachable, the turn waits in `~/.cem888/cowork/pending_*.json` and is committed exactly once on the next Stop or session start (same turn id, idempotent replay). Set `CEM888_SYNC_EXHALE=1` to commit inline instead.
- Timeouts: `CEM888_BEGIN_TIMEOUT` (default 8s), `CEM888_FINISH_TIMEOUT` (default 45s).
- The prohibition gate covers every tool call that passes through the host's PreToolUse hook. Matching is literal/typed; semantic equivalents that avoid the listed strings are not caught (advisory layer via the skill only).
- Logs: `~/.cem888/cowork/hooks.log`.
- The mid-work refresh fails soft: a transient CEM outage mid-turn emits nothing rather than a status message on every tool call.
- Fail-soft never means fail-**silent** for a *credential* problem. Reachability and credential validity are classified separately: a transient outage stays quiet, but a rejected credential (HTTP 401/403) is reported **once per session** — at session start, or mid-work if that is where it is first seen — because a dead token is otherwise indistinguishable from a plugin that simply has nothing to say.
- `python3 hooks/scripts/cem_hook.py doctor` — on-demand credential check. Prints the endpoint, where the credential came from, and the real result of a `tools/call`. **`initialize` and `tools/list` return HTTP 200 even for a deliberately garbage token, so a 200 there proves nothing; only `tools/call` enforces the credential.** Exit codes: `0` healthy, `2` credential missing/invalid, `3` unreachable, `4` doctor crashed.
- Status: BETA. Not yet certified on the frozen customer artifact. Hooks in Cowork plugins have a reported Windows issue; test on each OS before claiming support.
