---
name: video-game-project
description: Use when orienting or planning work in this minimal AI Harness Cocos client, especially the Loading to Game to Welcome startup flow and safe project boundaries.
---

# AI Harness Minimal Client

## Project Shape

This is a Cocos Creator 3.8.8 minimal client for AI Harness certification. It is no longer a full interactive-video game project.

The only supported runtime path is:

```text
Loading.scene
  -> Loading.ts
  -> App.SwitchScene("Game")
  -> Game.scene
  -> Gaming.ts
  -> App.initialize()
  -> App.ShowWelcome()
```

## Required Runtime Scenes

- `assets/scene/Loading.scene`
- `assets/scene/Game.scene`

`Edit_scene.scene` and vscene/editor scenes are intentionally removed.

## Kept Client Areas

- `assets/script/funsan/core/`: reusable framework runtime.
- `assets/script/funsan/data/define/`: minimal data definitions still used by `AppData`.
- `assets/script/funsan/module/welcome/`: simple welcome window logic.
- `assets/resources/prefabs/welcome/`: welcome prefab.
- `assets/resources/prefabs/_preload_/`: registered common prefab windows.

## Removed Business Areas

Do not assume these exist or recreate them unless the certification scope changes:

- video runtime module
- legacy scene module
- editor module
- `module/0_util`
- `module/z_*win`
- business config tables
- editor prefabs
- vscene prefabs and videos
- planning, art, packaging, and resource-processing directories

## Safe Boundaries

- Keep `Loading -> Game -> Welcome` working before adding anything.
- Do not reintroduce business window names in `WindowName.ts` or `ResDefine.ts`.
- Keep `assets/resources` on a reference whitelist: retain only resources used by scenes, registered prefabs, or startup code.
- Core runtime files such as `App.ts`, `AppData.ts`, `AppPlatformSDK.ts`, `WindowManager.ts`, and `WindowBase.ts` are still high-risk and should be changed narrowly.
