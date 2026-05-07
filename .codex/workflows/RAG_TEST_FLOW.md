# 轻量 RAG 测试链路

本文用于验证外部 Harness、Codex Agent 或人工测试流程能否通过 RAG 检索到当前最小 Cocos 工程的正确上下文。根 `README.md` 是 GitHub 展示和 RAG 总入口，本文负责补充检索顺序、验证问题和命中判断。这里不实现 embedding、向量库、索引服务或客户端运行时代码。

Harness 的 Agent 架构流程见 `.codex/workflows/HARNESS_AGENT_FLOW.md`。本文只聚焦 RAG 检索链路和验证问题。

## 知识源

建议把下列文件和目录作为 RAG 知识源：

- `README.md`：仓库级入口、GitHub 展示页、最小运行链路、MCP/RAG/大型游戏开发模型说明入口。
- `AGENTS.md`：Agent 启动路由，说明应继续读取哪些规则文件。
- `.codex/AGENTS.md`：框架级 Harness / Skills 规则边界。
- `.codex/skills/`：可复用 Cocos Harness、窗口、Prefab、配置表和核心运行时规则。
- `cocos_game_testing/AI_PROJECT_OVERVIEW.md`：当前客户端项目概览和项目级路由。
- `cocos_game_testing/AGENTS.md`：客户端目录内的 Agent 启动规则。
- `cocos_game_testing/skills/`：当前最小客户端项目规则，尤其是启动链路和安全边界。

## 推荐检索顺序

```text
README.md
  -> AGENTS.md
  -> .codex/AGENTS.md
  -> cocos_game_testing/AGENTS.md
  -> cocos_game_testing/AI_PROJECT_OVERVIEW.md
  -> 按问题选择 .codex/skills/ 或 cocos_game_testing/skills/
  -> 必要时检索具体代码、Prefab 或资源路径
```

使用时优先让 RAG 找到“路由文档”，再进入具体 Skill 或代码。不要直接把旧业务关键词当成可恢复范围；当前工程以最小认证链路为准。

## 验证问题样例

| 问题 | 预期命中文件 | 预期答案关键词 |
| --- | --- | --- |
| 当前工程的最小启动链路是什么？ | `README.md`、`cocos_game_testing/AI_PROJECT_OVERVIEW.md`、`cocos_game_testing/skills/video-game-project/SKILL.md` | `Loading.scene -> Game.scene -> App.ShowWelcome()` |
| 新增普通窗口模块应该参考哪些规则？ | `.codex/skills/cocos-window-module/SKILL.md`、`.codex/skills/cocos-prefab-authoring/SKILL.md` | `WindowName`、`ResDefine`、`WindowBase`、`XComponent`、`XButton`、`720 x 1280` |
| 当前哪些旧业务模块不应恢复？ | `cocos_game_testing/AI_PROJECT_OVERVIEW.md`、`cocos_game_testing/skills/video-game-project/SKILL.md` | `vscene`、`Edit_scene`、`module/z_*win`、旧业务配置和资源 |
| `assets/resources` 的保留原则是什么？ | `README.md`、`cocos_game_testing/skills/video-game-project/SKILL.md` | 引用白名单、启动链路、欢迎界面、公共窗口 |
| Cocos MCP 或 `cocos-cli` 是运行时依赖吗？ | `README.md`、`cocos_game_testing/AI_PROJECT_OVERVIEW.md`、`.codex/skills/cocos-project-harness/SKILL.md` | 可选测试工具、不是客户端运行时依赖、不提交工具源码 |
| RAG 在当前工程里实现了吗？ | `README.md`、本文档 | 文档级测试链路、外部消费者、不实现向量库或 embedding |
| Planner / Generator / Evaluator 在本 Harness 中分别负责什么？ | `.codex/workflows/HARNESS_AGENT_FLOW.md` | Planner 产出计划、Generator 生成变更、Evaluator 验证结果 |
| Agent 进入仓库后应该先读什么？ | `AGENTS.md`、`.codex/AGENTS.md`、`cocos_game_testing/AGENTS.md` | 入口路由、框架级规则、项目级规则 |
| 项目级问题和框架级问题如何分流？ | `.codex/AGENTS.md`、`cocos_game_testing/AI_PROJECT_OVERVIEW.md` | 框架规则进 `.codex/skills/`，项目规则进 `cocos_game_testing/skills/` |

## 判断标准

- 回答能引用或命中上表中的预期文件。
- 回答能区分 MCP、RAG 和 Cocos 客户端运行时：MCP 是可选工具接入，RAG 是外部文档检索链路，客户端仍只保留最小启动链路。
- 回答不会把已删除的旧业务模块、策划/美术/工具链目录或运行时资源重新视为当前认证范围。
- 回答新增窗口或 Prefab 时，会优先命中窗口/Prefab Skill，而不是凭空生成并行窗口系统。

## 非目标

- 不在仓库内生成向量索引、数据库、embedding 缓存或检索服务。
- 不把 `cocos-cli`、MCP 服务、node_modules 或外部工具源码提交进仓库。
- 不改变 `Loading -> Game -> Welcome` 最小启动链路。
