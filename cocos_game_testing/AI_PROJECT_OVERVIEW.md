# AI Harness Client Overview

> 更新时间：2026-05-06  
> 适用工程：`cocos_game_testing/`  
> 目的：认证 AI Harness 的最小 Cocos Creator 3.8.8 客户端工程。

## 一句话结论

这是一个 **Cocos Creator 3.8.8 + TypeScript** 的最小客户端工程，只用于验证 Harness 结构、AGENTS 路由、Skills 分层和基础启动链路。

旧游业务模块已经移除：不再保留 `vscene`、`Edit_scene`、`module/z_*win`、策划/美术/工具链输入目录或业务配置资源。

## 启动主线

```text
Loading.scene
  -> Loading.ts
  -> App.SwitchScene("Game")
  -> Game.scene
  -> Gaming.ts
  -> App.setAndInitStage()
  -> App.initialize()
  -> App.ShowWelcome()
```

## 保留内容

- `assets/scene/Loading.scene`
- `assets/scene/Game.scene`
- `assets/script/funsan/core/`
- `assets/script/funsan/data/define/`
- `assets/script/funsan/module/welcome/`
- `assets/resources/prefabs/welcome/main_welcome_ui.prefab`
- `assets/resources/prefabs/_preload_` 中注册的公共窗口 prefab
- 启动、欢迎和公共窗口直接引用的最小资源

## 已移除内容

- `Edit_scene.scene`
- video runtime module
- legacy scene module
- editor module
- `module/0_util`
- 所有 `module/z_*win`
- 旧业务 prefab、视频、配置表、编辑器资源和未引用素材

## 项目级 Skill

- `skills/video-game-project/SKILL.md`：描述当前最小客户端边界和启动链路。

框架级规则仍在 `../.codex/skills/`，例如 `cocos-project-harness`、`cocos-core-runtime` 和 `cocos-prefab-authoring`。

## RAG 可检索知识源

轻量 RAG 测试说明见 `../.codex/workflows/RAG_TEST_FLOW.md`。RAG 的消费者是外部 Harness、Codex Agent 或人工测试流程，不是 Cocos 客户端运行时代码。

- 项目级边界、启动链路和已移除内容：优先检索 `skills/video-game-project/SKILL.md`。
- 窗口模块新增、注册和生命周期：优先检索 `../.codex/skills/cocos-window-module/SKILL.md`。
- Prefab 结构、绑定、XButton 和 720 x 1280 规则：优先检索 `../.codex/skills/cocos-prefab-authoring/SKILL.md`。
- 仓库入口和读取顺序：检索 `../README.md`、`../AGENTS.md`、`AGENTS.md`。

## 可选 Cocos MCP 测试

本工程同时用于 Harness、Cocos MCP 和 RAG 流程验证。`cocos-cli` 是本机测试工具，不是客户端运行时依赖，也不提交到仓库。

当前推荐启动方式：

```powershell
cocos.cmd start-mcp-server --project E:\projects\fanxingame\game_project\cocos_game_testing\cocos_game_testing\cocos_game_testing --port 9527
```

Codex MCP 注册地址：

```text
http://127.0.0.1:9527/mcp
```

注意：在 PowerShell 中优先使用 `cocos.cmd`，避免 `cocos.ps1` 被执行策略拦截。
