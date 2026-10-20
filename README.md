# universal-modder-hwa

> [!IMPORTANT]
> **이 저장소는 원작 저장소가 아닙니다.**
> 원작은 Rehan과 기여자들이 만든 [rehan-remade/universal-modder](https://github.com/rehan-remade/universal-modder)(MIT)이고,
> 이 저장소는 원작의 핵심 로직만 가져와 개인 용도(안티그래비티 + Claude Code)로 고친 **비공식 포크**입니다.
> 원작자와 관계가 없으며, 원작자의 검토나 지원을 받지 않습니다.
> 버그 제보, 기능 요청, 지식 베이스 기여처럼 원작에 관한 일은 [원작 저장소](https://github.com/rehan-remade/universal-modder)를 이용해 주세요.
>
> **This is not the original repository.** It is an unofficial, personal fork of
> [rehan-remade/universal-modder](https://github.com/rehan-remade/universal-modder) with the fal integration removed.
> It is not affiliated with or endorsed by the original authors. Please report upstream issues there.

## 원작과 다른 점
- **fal 관련 기능을 전부 뺐습니다:** `um fal` CLI, fal MCP 서버, fal-assets 스킬, `FAL_KEY` 처리.
- **안티그래비티와 Claude Code 설정만 남겼습니다.** Codex, Cursor, VS Code/Copilot, Gemini CLI 확장, OpenCode 설정은 없습니다.
- **예제(`examples/`), 홍보용 이미지, GitHub 워크플로는 가져오지 않았습니다.** 스킬에 나오는 `examples/...` 경로는 원작 저장소에서 보시면 됩니다.
- **플러그인 이름은 `universal-modder-hwa`입니다.** 원작 플러그인과 같이 설치해도 이름이 겹치지 않습니다.
- 원작 커밋 `23902ef`(2026-10) 기준이며, 원작 업데이트를 자동으로 따라가지 않습니다.

## 들어 있는 것
| 경로 | 내용 |
|---|---|
| `um/`, `bin/um`, `bin/um.cmd` | `um` CLI: `scan`, `comfy`, `sprite`, `render3d`, `video`, `win`, `backup`, `publish`, `kb` |
| `skills/` | 에이전트 스킬 (사본: `.agents/skills` = Antigravity, `.claude/skills` = Claude Code) |
| `agents/` | Claude Code용 3인 팀 (orchestrator / game-worker / offline-worker) |
| `hooks/`, `.claude/settings.json` | 세션 시작 시 `um`을 PATH에 넣는 훅 |
| `knowledge/` | 원작에서 가져온 게임별 모딩 필드 노트 (`um kb search`) |
| `plugin.json`, `.claude-plugin/plugin.json` | Antigravity / Claude Code 플러그인 매니페스트 |
| `.claude-plugin/marketplace.json` | Claude Code 마켓플레이스 (`claude plugin marketplace add https://github.com/magjangin/universal-modder-hwa` → `claude plugin install universal-modder-hwa@universal-modder-hwa`) |

## 빠른 시작
```bash
uv run --with pytest pytest -q tests
```
```bash
bin/um kb search "Unity"
```
Windows cmd/PowerShell에서는 `bin` 폴더를 PATH에 넣고 `um ...`으로 실행합니다(`bin\um.cmd`).

에이전트용 상세 안내와 원작 대비 변경 이유는 [AGENTS.md](AGENTS.md)에 있습니다.

## 라이선스와 출처
원작과 같은 MIT 라이선스입니다. [LICENSE](LICENSE)의 원작 저작권 표시(Copyright (c) 2026 Rehan and universal-modder
contributors)를 그대로 유지합니다. `knowledge/`의 필드 노트는 원작 기여자들이 쓴 것입니다.
