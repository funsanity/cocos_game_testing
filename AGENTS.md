# Agent Startup

本仓库是 AI Harness 认证用的最小 Cocos 工程。开始任务前请按目录作用域读取：

- `.codex/AGENTS.md`：框架级 Harness / Skills 规则。
- `cocos_game_testing/AGENTS.md`：当前最小客户端项目规则。

如任务涉及 RAG、检索链路或 Harness 认证材料验证，可参考 `.codex/workflows/RAG_TEST_FLOW.md`；普通代码任务不需要默认读取该文档。

## Scope

- 根目录 `AGENTS.md` 只做入口路由，不放具体框架或项目实现规则。
- 框架级说明放在 `.codex/AGENTS.md` 和 `.codex/skills/`。
- 项目级说明放在 `cocos_game_testing/AGENTS.md`、`cocos_game_testing/AI_PROJECT_OVERVIEW.md` 和 `cocos_game_testing/skills/`。
- 如果规则冲突，以更靠近目标文件的 AGENTS 为准。
