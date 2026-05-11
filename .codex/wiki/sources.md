# Karpathy Wiki Sources

本文记录 Karpathy Wiki 的 Raw Sources。Raw Sources 是可信源材料，Wiki 页面负责整理和沉淀理解，不替代这些源文件。

## Core Routing Sources

| Source | Role | Priority |
| --- | --- | --- |
| `README.md` | 仓库总入口、GitHub 展示、Harness/RAG/MCP/大型游戏开发模型说明 | required |
| `AGENTS.md` | 仓库级 Agent 启动路由 | required |
| `.codex/AGENTS.md` | 框架级 Harness / Skills 边界 | required |
| `cocos_game_testing/AGENTS.md` | 当前最小客户端项目启动规则 | required |
| `cocos_game_testing/AI_PROJECT_OVERVIEW.md` | 当前客户端概览、启动链路、RAG 可检索知识源 | required |

## Workflow Sources

| Source | Role | Priority |
| --- | --- | --- |
| `.codex/workflows/RAG_TEST_FLOW.md` | 轻量 RAG 检索链路、验证问题、命中判断 | required-for-rag |
| `.codex/workflows/HARNESS_AGENT_FLOW.md` | RAG Context / Planner / Generator / Evaluator 协作模型 | required-for-harness |

## Skill Sources

| Source | Role | Priority |
| --- | --- | --- |
| `.codex/skills/cocos-project-harness/SKILL.md` | Cocos Harness 基线、两场景链路、目录边界、环境检查 | required |
| `.codex/skills/cocos-core-runtime/SKILL.md` | App、Global、Manager、XComponent、核心运行时规则 | required-for-runtime |
| `.codex/skills/cocos-window-module/SKILL.md` | 新增普通窗口模块、WindowName、ResDefine、WindowBase、生命周期 | required-for-window |
| `.codex/skills/cocos-prefab-authoring/SKILL.md` | Prefab 节点、绑定、Widget、XButton、720 x 1280 UI 规则 | required-for-prefab |
| `.codex/skills/cocos-config-table/SKILL.md` | 配置表、导出、注册、gameconfig 流程 | optional-currently |
| `cocos_game_testing/skills/video-game-project/SKILL.md` | 当前最小客户端边界、Loading -> Game -> Welcome、资源保留原则 | required |

## Project Config Sources

| Source | Role | Priority |
| --- | --- | --- |
| `cocos_game_testing/package.json` | 项目依赖和脚本 | required-for-env |
| `cocos_game_testing/tsconfig.json` | TypeScript 编译配置 | required-for-ts |
| `cocos_game_testing/cocos.config.json` | Cocos 项目配置 | required-for-cocos |

## Framework Runtime Sources

| Source | Role | Priority |
| --- | --- | --- |
| `cocos_game_testing/assets/script/funsan/core/**/*.ts` | 框架基础源码，包含 App、Global、AppData、AppPlatformSDK、Manager、工具类、基础组件和窗口系统 | framework-source |

`core/**/*.ts` 应沉淀为框架理解页面，例如 `core/runtime-overview.md`、`core/app-and-startup.md`、`core/window-system.md`、`core/component-conventions.md`、`core/resource-sound-event-managers.md`、`core/platform-persistence.md` 和 `core/high-risk-files.md`。不要把 Wiki 写成逐函数 API 文档。

## Runtime Entry Sources

| Source | Role | Priority |
| --- | --- | --- |
| `cocos_game_testing/assets/scene/Loading.scene` | 启动场景 | required-for-runtime |
| `cocos_game_testing/assets/scene/Game.scene` | 游戏主场景 | required-for-runtime |
| `cocos_game_testing/assets/script/funsan/core/view/component/Loading.ts` | Loading.scene 入口脚本 | required-for-runtime |
| `cocos_game_testing/assets/script/funsan/core/view/component/Gaming.ts` | Game.scene 入口脚本 | required-for-runtime |
| `cocos_game_testing/assets/script/funsan/core/App.ts` | App 初始化、场景切换、欢迎窗口入口 | required-for-runtime |

## Feature And Prefab Sources

| Source | Role | Priority |
| --- | --- | --- |
| `cocos_game_testing/assets/script/funsan/module/welcome/**/*.ts` | 欢迎窗口模块 | required-for-welcome |
| `cocos_game_testing/assets/resources/prefabs/welcome/main_welcome_ui.prefab` | 欢迎界面 prefab | required-for-welcome |
| `cocos_game_testing/assets/resources/prefabs/_preload_/*.prefab` | 公共预加载窗口 prefab | required-for-window-runtime |
| `cocos_game_testing/assets/script/funsan/module/signin/**/*.ts` | 每日签到认证示例模块 | query-when-needed |
| `cocos_game_testing/assets/resources/prefabs/signinview/DailySignIn_Win_fab.prefab` | 每日签到示例窗口 prefab | query-when-needed |
| `cocos_game_testing/assets/script/funsan/module/ranking/**/*.ts` | 排行榜认证示例模块 | query-when-needed |
| `cocos_game_testing/assets/resources/prefabs/rankingview/Ranking_Win_fab.prefab` | 排行榜示例窗口 prefab | query-when-needed |

## Maintenance Rule

新增稳定知识时，优先判断它属于 Raw Source、Skill、workflow 还是 Wiki 页面。Raw Sources 保持可信来源身份，Wiki 页面负责整理理解并指回来源。
