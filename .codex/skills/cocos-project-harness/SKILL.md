---
name: cocos-project-harness
description: Use when orienting, creating, auditing, or standardizing a Cocos Creator 3.8.8 project that should follow this reusable two-scene Harness, environment-check, design-resolution, and framework/project skill split.
---

# Cocos Project Harness

## Core Baseline

Use this skill as the framework-level entry for Cocos Creator 3.8.8 projects that follow this Harness. Keep rules here reusable across projects; put project-specific gameplay, content, editor, and asset rules in the project's own `skills/` folder.

## Standard Repository Layout

These top-level directories are part of the Harness standard. Keep the names stable for new projects unless the user explicitly chooses a different layout and all tool references are updated.

| Path | Framework responsibility |
| --- | --- |
| `cocos_game_testing/` | Cocos Creator client project. Runtime scenes, scripts, resources, prefabs, settings, and client config live here. |
| `cocos_game_testing_server/` | Server project or server placeholder. Do not assume it is implemented; check before server work. |
| `cehua/` | Planning input: design documents, scripts, chapter/level plans, source video/story materials. |
| `meishu/` | Art input: source art, UI mockups, sliced images, effect images, and art handoff files. |
| `tool/` | Tool source: export scripts, resource processors, texture packing, audio/video processing, packaging helpers. |
| `toolp/` | Packaged tools and external utilities: built executables, upload/browser tools, and vendor tool payloads. |

The common production flow is:

```text
cehua + meishu
  -> tool / toolp
  -> cocos_game_testing assets, config, resources, or build outputs
```

When paths change, inspect tool scripts before moving files because many tools may use relative paths.

## Required Startup Shape

- Required runtime scenes are `Loading.scene` and `Game.scene`.
- `Loading.scene` loads the minimum resources needed to enter the game.
- `Game.scene` owns internal game initialization and runtime logic.
- Extra scenes such as `Edit_scene.scene` are optional project-level tools. Do not treat them as required Harness scenes.
- The expected startup path is: `Loading.scene -> Loading.ts -> App.SwitchScene("Game") -> Game.scene -> Gaming.ts -> App.initialize()`.

## Skill Placement

- Put reusable framework rules under `.codex/skills/`.
- Put project-specific rules under `<project>/skills/`.
- Keep overview docs as route maps. Move detailed workflows into Skills.
- When a task could match both layers, read the framework skill first, then the project skill.

## UI And Adaptation Defaults

- Design new or modified runtime UI against `720 x 1280`.
- Keep ordinary window root nodes at `720 x 1280`.
- Do not add `cc.Widget` to root window nodes by default.
- Use `cc.Widget` on internal nodes that must adapt to parent or screen edges.
- Use `FitSizeComp` or the project's equivalent adaptation component when screen/device adaptation is required.
- All components or elements with color design default to `ffffffff`; change color only for explicit states such as selected, disabled, mask, warning, or effect.

## Environment Checks

Before running build, export, validation, or code-generation work, check the required local environment instead of assuming it exists.

- Check only what the task needs: Cocos Creator, Node/npm, TypeScript, Python, Excel/export dependencies, PowerShell script policy, project dependencies, and local tools.
- Prefer low-risk checks such as `node -v`, `npm -v`, `tsc -v`, path existence, and script existence.
- If a required tool is missing, say what is missing, which workflow is blocked, and what the user should install or configure.
- Do not silently skip verification because the environment is missing.
- If installing requires network access, global package changes, or elevated permissions, ask the user before installing.

## Optional cocos-cli

`cocos-cli` (`https://github.com/cocos/cocos-cli`) is an optional future Harness tool, not a required dependency.

Potential uses:

- create/import Cocos projects for new Harness work
- run command-line project checks or builds
- assist resource/project automation
- start the optional MCP server for AI-assisted Cocos workflows

Rules:

- Check whether `cocos` / `cocos-cli` is installed before using it.
- If it is missing, report that it is optional and currently unavailable; do not treat the task as failed unless the task specifically requires it.
- Do not install it without user approval, because installation may require network access, global package changes, native build tools, or PATH updates.
- Do not replace existing project `tool/`, `toolp/`, export, packaging, or prefab-generation workflows with `cocos-cli` until compatibility is tested.
- Only use `cocos-cli` or its MCP server as a prefab node/component editing tool after real testing confirms it can safely edit Cocos Creator prefab structure and bindings.

## Editing Policy

- Do not clean historical code globally when standardizing the Harness.
- Apply these rules to new or modified files.
- If current code conflicts with a new Harness rule, document the current state and the forward rule; do not rewrite runtime logic unless the task asks for it.

## Common Mistakes

- Treating optional editor scenes as required runtime scenes.
- Putting project gameplay rules into `.codex/skills/`.
- Duplicating long project details in `AI_PROJECT_OVERVIEW.md` instead of routing to Skills.
- Skipping environment checks and reporting success without evidence.
