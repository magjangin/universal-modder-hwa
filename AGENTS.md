# universal-modder-hwa

A game-modding toolkit and a shared knowledge base for AI coding agents, trimmed for **Antigravity** and
**Claude Code**. It is an unofficial fork (not the original repository) of [rehan-remade/universal-modder](https://github.com/rehan-remade/universal-modder)
(MIT) with everything fal (the `um fal` CLI, the fal MCP server, the fal-assets skill, `FAL_KEY` handling)
removed. When someone opens an agent in this repo, they almost always want to **mod a game**, or to learn how
a game was modded.

## Start here
1. **Read `skills/mod-any-game/SKILL.md` and follow its loop:** intake → recon → route → lab (backups) →
   source of truth → vertical slice → assets → verify in the real game → showcase → publish.
2. **Search the knowledge base first:** `bin/um kb search "<game or engine>"`. Other agents may already have
   written down the exact versions, routes and gotchas (`knowledge/INDEX.md`).
3. **At the end, write down what you learned:** a field note (`bin/um kb new ...`). `um kb pr` opens a PR
   against the upstream repo (`UM_KB_REPO` overrides it); only run it when the human agrees.

## Tools
- **`bin/um`** is the Python CLI, and it sets itself up with `uv`.
  - On Windows, add `bin` to PATH and `bin\um.cmd` runs it from cmd or PowerShell; from Git Bash,
    `export PATH="$PWD/bin:$PATH"`. Or install it: `uv tool install <this repo's folder>`.
  - Every group has `--help`:
    - `scan`: installed games, engine, anti-cheat, loaders, saves, routes
    - `comfy`: images from a local ComfyUI server, with no API key
    - `sprite` / `render3d`: art → engine-ready frames (`render3d` needs Blender on PATH or `BLENDER=...`)
    - `win`: launch, screenshot, input, record on Windows (also from WSL)
    - `video`: contact sheets and EDL showcase edits
    - `backup`: snapshot and restore saves
    - `publish`: pre-release lint
    - `kb`: the knowledge base
- **No MCP server is bundled.** Assets come from the user, from a local ComfyUI (`um comfy`), or from
  whatever generator the user chooses; record where each asset came from in `MODLOG.md`.
- **Skills** (`skills/*/SKILL.md`, Agent Skills format) are copied where the agents look for them:
  `.agents/skills` (Antigravity) and `.claude/skills` (Claude Code). Edit `skills/`, then copy it over
  (`rm -rf .agents/skills .claude/skills && cp -r skills .agents/skills && cp -r skills .claude/skills`);
  a test fails while the copies differ.
- **Engine playbooks:** `skills/mod-any-game/references/engines/`.
- **Worked examples** (`examples/...` in the skills) were not copied. Read them upstream:
  https://github.com/rehan-remade/universal-modder/tree/main/examples

## Rules (full reasoning in `skills/mod-any-game/references/safety.md`)
- **What you can mod:** any game the user owns: single-player, multiplayer, or servers the user hosts.
  - Never touch online clients protected by anti-cheat.
  - Never write cheats against other players (aimbots, ESP, speed hacks).
  - Never bypass anti-cheat, DRM or ownership checks.
- **Saves:** `um backup` saves before modded launches.
- **What you ship:** never commit or publish game files, extracted assets or decompiled code. Keep
  decompiles outside the repo.
- **Processes:** kill by exact PID (`um win kill`), never by name pattern.
- **Ask first** before:
  - driving the user's mouse and keyboard, and never focus a game with an online mode while they're typing;
  - installing loaders into game folders or changing the registry;
  - publishing anything, PRs included.
- **Keep a journal:** a `MODLOG.md` in the mod's working folder. It becomes your field note at the end.

## Working on the toolkit itself
- **Python:** 3.10+, deps Pillow, numpy and PyYAML (`uv` handles them via `bin/um`). Code lives in `um/`,
  one module per CLI group, each with `register(sub)` and a docstring that doubles as `--help`. Groups are
  listed in `um/cli.py` (`GROUPS`).
- **Windows tools:** `um/ps1/*.ps1` embed C# 5 (Windows PowerShell 5.1's compiler): no string
  interpolation, no `out var`, no expression-bodied members.
- **Tests:** `uv run --with pytest pytest -q tests`. Also run `um kb check --index` and `um publish check .`.
- **Wording:** keep skills and docs agent-neutral ("the agent", not a product name) except in sections
  about one specific agent.

## Fork notes (what changed from upstream, and why)
- Forked from upstream commit `23902ef` (2026-10). Not tracking upstream automatically; when pulling new
  upstream work, re-apply the removals below.
- **Removed:** `um/fal.py`, `skills/fal-assets`, `.mcp.json` / `mcp.json` (fal MCP only), `tests/test_fal_key.py`,
  the fal tests in `tests/test_um.py`, and the configs for agents not used here (Codex, Cursor, VS Code /
  Copilot, Gemini CLI extension, OpenCode plugin), plus `examples/`, `docs/media`, README/CONTRIBUTING and
  the GitHub workflows.
- **`um comfy image --sprite`** used to import `SPRITE_STYLE` from `um.fal`; the string now lives in
  `um/comfy.py`.
- **`um publish check`** no longer looks for fal keys or a `fal_manifest.jsonl` credit line; the other
  secret patterns (Anthropic, OpenAI, GitHub, AWS...) are unchanged, and the tests use an Anthropic-style
  fake key instead.
- **Plugin name is `universal-modder-hwa`** (`plugin.json` for Antigravity, `.claude-plugin/plugin.json` for
  Claude Code) and the PATH hook marker is `universal-modder-hwa-path`, so it can sit next to an installed
  upstream plugin without one silently shadowing the other. The Python package is still `um`, so whichever
  `bin/` comes first on PATH wins.
- **`.claude-plugin/marketplace.json`** makes this repo its own Claude Code marketplace, also named
  `universal-modder-hwa`: `claude plugin marketplace add https://github.com/magjangin/universal-modder-hwa`, then
  `claude plugin install universal-modder-hwa@universal-modder-hwa`. Without it `marketplace add` fails
  ("Marketplace file not found"). Mashup Prompter (magjangin/mashup-mods-tool) installs it this way.
