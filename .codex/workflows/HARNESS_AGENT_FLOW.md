# Harness Agent 架构流程

本文描述当前 AI Harness 认证工程的轻量 Agent 协作模型。它是认证/演示用的架构说明，不实现多 Agent 服务，不创建运行时代码，也不要求每个任务都拆成三个独立会话。

## 总体链路

```text
User Request
  -> RAG Context
  -> Planner
  -> Generator
  -> Evaluator
  -> Result / Revision
```

这条链路用于说明 Agent 如何读取上下文、制定计划、生成变更并验证结果。当前工程仍保持最小 Cocos 客户端运行链路：`Loading -> Game -> Welcome`。

大型游戏开发模型的 GitHub 展示总览见根 `README.md`；本文只定义 Harness 内部的 RAG、Planner、Generator 和 Evaluator 协作方式。

## RAG Context

RAG 是上下文输入层，不是 Cocos 客户端运行时依赖。它负责从仓库文档和必要代码中取回任务相关上下文，交给后续流程角色使用。

推荐知识源：

- `README.md`：仓库入口、最小运行链路、MCP/RAG 测试入口。
- `AGENTS.md`：启动路由和作用域边界。
- `.codex/AGENTS.md` 与 `.codex/skills/`：框架级 Harness、窗口、Prefab、配置表和核心运行时规则。
- `cocos_game_testing/AI_PROJECT_OVERVIEW.md` 与 `cocos_game_testing/skills/`：当前最小客户端项目边界。
- 必要时检索具体代码、Prefab、资源和配置路径。

RAG 只提供可引用的上下文，不负责直接改文件，也不替代 Planner 的判断。

## 三个流程角色

### Planner

Planner 负责把用户需求转成可执行计划：

- 读取 RAG 命中的入口文档、AGENTS、Skills 和必要代码。
- 明确目标、范围、约束、假设和验收标准。
- 产出能交给 Generator 执行的计划。
- 对不清楚的产品意图或高风险取舍先提问，不直接猜测。

### Generator

Generator 负责按计划生成或修改工程内容：

- 修改文档、TypeScript、Prefab、资源、配置或脚本。
- 遵守项目边界，不恢复已删除旧业务模块。
- 新增窗口、Prefab 或资源时优先遵循对应 Skill。
- 保持输出可被 Evaluator 验证。

### Evaluator

Evaluator 负责独立检查输出是否满足计划：

- 按 Test Plan 执行引用、路径、资源、类型和范围检查。
- 验证 `Loading -> Game -> Welcome` 最小启动链路没有被破坏。
- 检查是否误引入运行时代码、旧业务资源、node_modules、索引文件或工具源码。
- 发现问题时把结果反馈给 Planner 或 Generator 进入修订。

## 运行方式

默认运行方式是 **单会话串行角色**：同一个 Agent 依次完成 RAG Context、Planner、Generator 和 Evaluator。这适合当前最小认证工程，也最容易保持上下文一致。

复杂任务可以使用 **subagent 方式**：

- 主会话担任 Planner，负责目标、范围和计划。
- worker subagent 担任 Generator，处理明确的代码、Prefab 或文档改动。
- explorer/reviewer subagent 担任 Evaluator，做独立检查或风险审查。

只有在认证演示或评测需要隔离职责时，才模拟 **三个独立会话**。三会话模式能证明计划可交接、生成可复现、评估可独立，但需要额外管理上下文同步和文件状态。

## 非目标

- 不实现多 Agent 编排服务。
- 不新增向量库、embedding 服务、数据库或索引产物。
- 不把 RAG、MCP 或 Agent 流程作为 Cocos 客户端运行时依赖。
- 不改变现有 AGENTS / Skills 的职责边界。
