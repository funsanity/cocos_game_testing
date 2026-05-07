---
name: cocos-core-runtime
description: Use when working with this Cocos framework runtime core, including App, Global, AppData, AppPlatformSDK, managers, events, util classes, base classes, XComponent, XButton, and persistent data conventions.
---

# Cocos Core Runtime

## Core Principle

Treat `assets/script/funsan/core/` as the framework runtime. Business modules should call the core through stable public APIs instead of recreating platform, event, resource, window, sound, or frame-loop systems.

## Runtime Entry Points

- `App.ts` is the application hub. It owns layers, scene switching, window entry helpers, managers, and high-level game services.
- `Global.ts` is shared global access. Avoid adding business state here unless it is truly framework-wide.
- `AppData.ts` is the persistent data and runtime state boundary.
- `AppPlatformSDK.ts` is the platform bridge around `window.__CONFIG__`.
- Managers such as `FrameTimeManager`, `ResourceManager`, `SoundManager`, `WindowManager`, `EventManager`, `ReconnectManager`, and `RadioGroupManager` are framework services.

## Component Rules

- New custom Cocos node components must extend/use `XComponent` unless they are engine-level exceptions.
- Prefab buttons must mount/use `XButton`.
- Do not create parallel button, event, window, sound, or resource managers for ordinary modules.
- Public Inspector fields must have Chinese comments/tooltips.
- Important UI nodes must be exposed as script properties and mounted in the prefab; avoid runtime name/path lookup for new UI.

## XComponent Rules

- `XComponent` provides `this.app = App.Instance()` and automatic cleanup for event/UI bindings.
- If a subclass overrides `onLoad`, `start`, `registerEvents`, `removeEvents`, or `onDestroy`, call the matching `super.*` unless matching a deliberate legacy exception.
- Register UI events through `addUIEvent(...)` when the event should be automatically removed on disable/destroy.
- `removeEvents()` calls `this.app.targetOff(this)` and clears UI events registered by `addUIEvent(...)`; do not duplicate cleanup unless the subclass owns extra external listeners.
- Prefer `Node.EventType.TOUCH_*` or Cocos component event constants consistently with nearby modules.

## XButton Rules

- `XButton` extends `cc.Button` and already reports button behavior plus plays the normal click sound on touch start.
- Use `XButton` for prefab buttons so click sound/reporting behavior stays centralized.
- Do not replace `XButton` with raw `cc.Button` in new prefabs unless the user explicitly asks for a special engine-level exception.

## FitSizeComp Rules

- `FitSizeComp` is the project adaptation component for the `720 x 1280` portrait baseline.
- Use it, or a project-approved equivalent, when a screen needs device/resolution adaptation.
- Do not put root `cc.Widget` on every window to solve adaptation; use internal `Widget` nodes and `FitSizeComp` where appropriate.
- Be careful when editing `FitSizeComp`: it changes Cocos `view` design resolution policy and browser/mobile resize behavior.

## Manager Usage Rules

- `FrameTimeManager.addFrameHamdler(...)` is the frame-loop scheduler. Use stable unique keys, remove handlers when no longer needed, and set `bRemWhenSwitchScene=false` only for handlers that must survive scene switches.
- `EventManager` extends Cocos `EventTarget`. Use `app.dispatch(...)`, `app.eventCenter.emit(...)`, or the existing wrapper style; rely on `targetOff` cleanup for target-bound events.
- `WindowManager.CreateWindow(...)` has an internal `openRegister` parameter. Business code should call `this.app.CreateWindow(name, parent, ...arg)` instead of calling `windowMgr.CreateWindow` directly.
- `ResourceManager` owns `resources.load`, `resources.loadDir`, prefab lookup, atlas image setting, and load-timeout checks. Reuse it instead of duplicating resource loaders in modules.
- `SoundManager` owns BGM, SFX, and managed sound playback. Use `playManagedSound` / `stopManagedSound` for flow-controlled sounds instead of unmanaged timers in business modules.

## AppData Conventions

- `_xxx` means a private backing value for a normal persistent public property. Load it with `GetLocal(...)` and save it through the public setter with `SetLocal(...)`.
- `__xxx` means a private bitmask or flag-set backing value, such as guide flags, daily reward flags, or claimed indexes.
- For `__xxx`, set flags with bit operations such as `value | (1 << index)` and check with `value & (1 << index)`.
- Business code should use public getters, setters, and helpers. Do not directly mutate private `_xxx` or `__xxx` fields.
- Add new persistent data conservatively. Check cross-day reset and platform sync implications before changing defaults.

## AppPlatformSDK Conventions

- Treat `AppPlatformSDK` as the only game-side platform bridge.
- Platform capability should be wrapped as an `AppPlatformSDK` method and call `window.__CONFIG__` through `GetConfigVal` or `CallConfigFun`.
- Business modules should not directly call `wx`, `tt`, `ks`, or scatter raw `window.__CONFIG__` reads.
- Do not rename existing `__CONFIG__` protocol keys unless the platform shell is updated too.
- Ad flow affects `AD_Win`, sound resume, `isAding`, and the `ad_complete` event. Review the full flow before changing ad callbacks.
- Login, share, rank, subscribe, desktop shortcut, upload, reload, and game-state callbacks may use different callback shapes; preserve the local convention.

## Events And Utils

- Use the existing event mechanism: `EventManager` and `GameEvent.ts` / `CommonEvents`.
- Reuse `MathUtils`, `Util`, `UtilTween`, `ConfigHelper`, and other `core/util` helpers before adding generic utilities.
- Put reusable framework helpers under `core`; put gameplay-specific helpers under the project module.

## High-Risk Files

Be careful with:

- `core/App.ts`
- `core/AppData.ts`
- `core/AppPlatformSDK.ts`
- `core/mgr/WindowManager.ts`
- `core/view/base/WindowBase.ts`

Small framework edits need targeted validation and a clear reason.

## Common Mistakes

- Adding business logic directly into `AppPlatformSDK`.
- Bypassing `AppData` setters and breaking persistence.
- Using raw `Component` for prefab UI where `XComponent` is required.
- Creating buttons without `XButton`.
- Adding global utilities that only one project module needs.
