# Hooks

Claude Code hooks that make some of the skills in this repo enforce or fire
automatically instead of relying on the model to remember them. All hooks fail
open (any error exits 0) and require `jq` and `bash`.

Copy the scripts to `~/.claude/hooks/` and merge the JSON blocks below into
`~/.claude/settings.json`.

## 1. no-superlative-guard

Enforces `no-sycophancy`, `no-invented-hazards`, `no-unmeasured-numbers`, and the
empty-deferral rule.

- **Stop**: scans the finished message; on a banned phrase it blocks the turn and
  forces a silent re-emit without it.
- **PostToolUse (Write|Edit)**: scans file content before it is written; exempts
  `~/.claude/**` since the pattern files quote the banned phrases by design.

Patterns live in `banned-patterns.sh` (six classes, documented inline) - tune the
regexes to your own habits.

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

## 2. handoff

Pairs with the `handoff` skill (`.claude/skills/handoff/`), which reads or writes
a session-handoff doc so a fresh session can continue the work. The three hooks
make it automatic, and only engage for sessions where the skill was actually used
(a per-session marker under `~/.claude/handoff-active/`):

- **handoff-mark.sh** - PreToolUse(Skill): when the handoff skill is invoked, marks
  the session active, so the other two hooks stay quiet in unrelated sessions.
- **handoff-load.sh** - SessionStart: injects the curated handoff doc (or the
  pre-compact safety net) as context so a resumed session starts current.
- **handoff-precompact.sh** - PreCompact: before compaction, captures firsthand
  recent state (git status, last commits, last user messages verbatim) as a
  safety net. Deliberately not a model summary, so nothing lossy is institutionalized.

```json
{
  "hooks": {
    "SessionStart": [
      { "hooks": [ { "type": "command", "command": "bash ~/.claude/hooks/handoff-load.sh" } ] }
    ],
    "PreCompact": [
      { "hooks": [ { "type": "command", "command": "bash ~/.claude/hooks/handoff-precompact.sh" } ] }
    ],
    "PreToolUse": [
      { "matcher": "Skill", "hooks": [ { "type": "command", "command": "bash ~/.claude/hooks/handoff-mark.sh" } ] }
    ]
  }
}
```

If you already have entries for these events, merge the arrays rather than
replacing them.
