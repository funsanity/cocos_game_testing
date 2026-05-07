# Cocos Harness Agent Startup

本目录是框架级 Harness / Skills 规则区，适用于可复用到后续 Cocos Creator 3.8.8 项目的通用流程。

## Required Reading

处理 `.codex/` 下的任何任务前，请先理解：

- `skills/cocos-project-harness/SKILL.md`
- `skills/cocos-core-runtime/SKILL.md`
- `skills/cocos-window-module/SKILL.md`
- `skills/cocos-config-table/SKILL.md`
- `skills/cocos-prefab-authoring/SKILL.md`

## Framework Scope

- 这里只放可跨项目复用的规则。
- 不要把当前影游项目的剧情、vscene、Edit_scene、具体资源命名或业务窗口写进框架级规则。
- 框架标准包括：固定顶层目录、`Loading.scene -> Game.scene`、Cocos 3.8.8、720x1280、环境检查、`XComponent`、`XButton`、`FitSizeComp`、核心 Manager、窗口系统、配置表流程和预制体通用规则。
- 固定顶层目录是 Harness 标准结构，不要当作临时杂项目录：`cocos_game_testing/`、`cocos_game_testing_server/`、`cehua/`、`meishu/`、`tool/`、`toolp/`。
- 框架层只规定这些目录的名称、职责和通用生产链路；具体剧本、美术、工具参数、服务端实现和客户端内容归项目层文档说明。
- 必要环境缺失时必须明确提醒，不要静默跳过验证或导出。

## Skill Rules

- 每个 skill 必须有标准 `SKILL.md`。
- frontmatter 只保留 `name` 和 `description`。
- `description` 必须以 `Use when` 开头，并描述触发场景。
- 新规则优先写到最匹配的 skill，不要堆进一个大文档。
- 更新 skill 后要做至少无依赖 frontmatter 检查；如果官方校验脚本因依赖缺失无法运行，要报告缺失环境。
