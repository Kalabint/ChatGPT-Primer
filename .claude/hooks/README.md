# no-superlative-guard

A Claude Code hook that enforces several of the behavioral skills in this repo
(`no-sycophancy`, `no-invented-hazards`, `no-unmeasured-numbers`, and the
empty-deferral rule) mechanically, instead of relying on the model to remember
them. The skills are the "what"; this hook is the "so it actually happens".

## What it does

- **Stop**: scans the assistant's finished message. On a banned phrase it blocks
  the turn and injects a terse correction instruction, so the model silently
  re-emits the same content without the phrase.
- **PostToolUse (Write|Edit)**: scans file content before it is written, so the
  banned phrasing never lands in a file either. It exempts `~/.claude/**`, since
  the pattern files quote the banned phrases verbatim by design.

Patterns live in `banned-patterns.sh` (six classes, documented inline). Tune the
regexes to your own tics. The guard fails **open**: any error exits 0, because a
hook that blocks the session is worse than a missed match.

## Requirements

- `jq` on PATH (the hook exits 0 if it is missing).
- `bash`.

## Install

Copy both scripts to `~/.claude/hooks/` and add this to `~/.claude/settings.json`:

```json
{
  "hooks": {
    "Stop": [
      { "hooks": [ { "type": "command", "command": "bash ~/.claude/hooks/no-superlative-guard.sh" } ] }
    ],
    "PostToolUse": [
      { "matcher": "Write|Edit", "hooks": [ { "type": "command", "command": "bash ~/.claude/hooks/no-superlative-guard.sh" } ] }
    ]
  }
}
```

`banned-patterns.sh` is sourced by the guard; it does not need its own hook entry.
