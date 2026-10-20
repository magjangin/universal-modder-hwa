@AGENTS.md

## Claude Code specifics
- Installed as a plugin, the skills are namespaced (`/universal-modder-hwa:mod-any-game`) and a SessionStart
  hook (`hooks/hooks.json`) puts `um` on PATH. No MCP server is bundled.
- In a clone, `.claude/settings.json` adds the same PATH hook, and `.claude/skills` is a copy of `skills/`.
- `agents/` holds a three-agent team for long mods: `mod-orchestrator`, run as the main session, spawns
  `mod-game-worker` and `mod-offline-worker`. Claude Code loads `agents/` only from a plugin, so in a clone
  start it with `claude --plugin-dir . --agent mod-orchestrator` (installed: `claude --agent mod-orchestrator`).
