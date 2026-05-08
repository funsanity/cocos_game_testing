---
name: cocos-window-module
description: Use when creating, editing, registering, or validating ordinary Cocos Creator window modules that use WindowName, ResDefine, WindowManager, WindowBase, XComponent, XButton, prefab-mounted nodes, scroll lists, and config-backed UI.
---

# Cocos Window Module

## Purpose

Create ordinary Cocos Creator 3.8 window modules using this Harness window pattern. Prefer existing project conventions over new architecture. If the current project has its own project-level window skill, read that after this framework skill.

## Hard Rules

- New runtime windows must use the existing window system: `WindowName`, `ResDefine`, `WindowManager`, and `WindowBase`.
- Do not create a standalone `Component` on a prefab and call it a window module.
- Do not create a parallel popup/window system.
- Do not edit `WindowManager.ts` or `WindowBase.ts` unless the user explicitly asks for core window-system changes.
- Prefab paths must be registered through `ResDefine.ts` and loaded by `WindowManager`.
- If a window has an independent `ViewComp`, the window class owns lifecycle forwarding in `winDidLoad`, `winDidShow`, `winWillHide`, and `winWillDestroy`.
- Buttons, tabs, and other interactions must not rely on passive component initialization only; the window entry should initialize/forward deliberately.
- New custom UI components must extend/use `XComponent`.
- Prefab buttons must mount/use `XButton`.
- Important UI nodes must be exposed as script properties and mounted in the prefab.

## UI Baseline

- Design ordinary window prefabs against a default resolution of `720 x 1280`.
- Keep the root window node at `720 x 1280`; do not add `cc.Widget` to the root node by default.
- Any component or element with color design should default to `ffffffff` unless a specific visual state requires another color.
- Add `cc.Widget` to nodes that must adapt to parent edges or screen size instead of relying only on fixed coordinates.
- Use `FitSizeComp` or the project equivalent when the window requires device/resolution adaptation.
- When a window contains `cc.ScrollView`, the `view` clipping node should have a `cc.Widget` that stretches to the same size as its parent scroll-view node.
- For `cc.ScrollView/view`, keep the clipping `cc.Mask` and `cc.Graphics` parameters consistent with working prefabs such as `OnlineReward_Win_fab`: `Mask._alphaThreshold = 0.1` and `Graphics._color.a = 255`. If `Graphics._color.a = 0` or the alpha threshold is too strict, items under `view/content` can be fully clipped even though the nodes are active and positioned correctly.

## Before Editing

1. Read the project overview and project-level skills when project context is needed.
2. Inspect a nearby module, especially:
   - `assets/script/funsan/module/z_secretarywin/`
   - another simple window under `assets/script/funsan/module/z_*win/`
   - `assets/script/funsan/core/define/WindowName.ts`
   - `assets/script/funsan/core/define/ResDefine.ts`
   - `assets/script/funsan/core/util/DataProvider.ts`
3. Confirm module name, entrance handler, config sheets, and whether it needs a scroll list/item prefab.

Ask the user if table field meaning is ambiguous. Otherwise make conservative assumptions and keep moving.

## Standard File Layout

For module `<Name>` and folder token `<module>`:

```text
assets/script/funsan/module/z_<module>win/
  <Name>Win.ts
  <Name>ViewComp.ts
  <Name>Data.ts
  <Name>ItemComp.ts        # only when list items are needed
```

Common examples:

```text
z_secretarywin/SecretaryWin.ts
z_secretarywin/SecretaryViewComp.ts
z_secretarywin/SecretaryData.ts
z_secretarywin/SecretaryItemComp.ts
```

## Window Class

`<Name>Win.ts` must inherit `WindowBase` and implement the project window lifecycle. When the prefab has a separate `<Name>ViewComp`, get it from `this.skin` in `winDidLoad()` and forward lifecycle calls:

```ts
import { _decorator } from "cc";
import WindowBase from "../../core/view/base/WindowBase";
import { <Name>ViewComp } from "./<Name>ViewComp";

const { ccclass } = _decorator;

@ccclass("<Name>Win")
export default class <Name>Win extends WindowBase {
    private viewComp: <Name>ViewComp | null = null;

    protected winDidLoad(): void {
        super.winDidLoad();
        this.viewComp = this.skin.getComponent(<Name>ViewComp);
        this.viewComp?.winDidLoad();
    }

    protected showWindowAction(): boolean {
        return true;
    }

    protected winDidShow(...arg: any[]): void {
        super.winDidShow(...arg);
        this.viewComp?.winDidShow(...arg);
    }

    protected winWillHide(): void {
        super.winWillHide();
        this.viewComp?.winWillHide();
    }

    protected winWillDestroy(): void {
        this.viewComp?.winWillDestroy();
        this.viewComp = null;
    }
}
```

Return `true` from `showWindowAction()` for normal popup mask, animation, and window layering behavior. If a nearby existing module has extra required cleanup, copy that local style conservatively.

## Window Argument Passing

Business modules should call the `App` wrapper, not the `WindowManager` signature directly.

Correct:

```ts
this.app.CreateWindow(WindowName.<Name>Win, null, data);
```

The receiving window should forward the first argument:

```ts
protected winDidShow(...arg: any[]): void {
    super.winDidShow(...arg);
    this.viewComp?.winDidShow(arg && arg.length > 0 ? arg[0] as <DataType> : null);
}
```

Why this matters:

- `App.CreateWindow(windowName, parent?, ...arg)` internally calls `WindowManager.CreateWindow(windowName, parent, this.openCallback, ...arg)`.
- `WindowManager.CreateWindow(windowName, parent?, openRegister?, ...arg)` has an extra `openRegister` parameter.
- Do not copy the `WindowManager` call shape into business code. `this.app.CreateWindow(WindowName.<Name>Win, null, null, data)` passes `[null, data]` to `winDidShow`, so `arg[0]` becomes `null` and the real config shifts to `arg[1]`.

Avoid:

```ts
this.app.CreateWindow(WindowName.<Name>Win, null, null, data);
```

Use direct `windowMgr.CreateWindow(...)` only in core framework code where the `openRegister` parameter is intentional.

## View Component Rules

Use mounted properties, not dynamic node lookup, for new module UI wiring.

Required style:

```ts
import { _decorator, Button, Label, Node } from "cc";
import { WindowName } from "../../core/define/WindowName";
import XComponent from "../../core/view/component/XComponent";

const { ccclass, property } = _decorator;

@ccclass("<Name>ViewComp")
export class <Name>ViewComp extends XComponent {
    @property({ type: Node, tooltip: "关闭按钮节点。" })
    public close_btn_node: Node | null = null;

    @property({ type: Label, tooltip: "标题文本。" })
    public title_txt: Label | null = null;

    public winDidLoad(): void {
        this.bindClick(this.close_btn_node, this.onCloseClick);
    }

    private onCloseClick(): void {
        this.app.HideWindow(WindowName.<Name>Win);
    }

    private bindClick(node: Node | null, handler: () => void): void {
        if (!node) {
            return;
        }
        if (!node.getComponent(Button)) {
            node.addComponent(Button);
        }
        node.off(Node.EventType.TOUCH_END, handler, this);
        node.on(Node.EventType.TOUCH_END, handler, this);
    }
}
```

Guidelines:

- Add `@property({ type, tooltip })` for every prefab-bound node/component.
- Every exposed property must have a clear Chinese tooltip/comment explaining the binding purpose, for example `tooltip: "关闭按钮节点。"` or `tooltip: "列表内容根节点。"`.
- Keep Chinese comments and tooltips encoded as UTF-8. If terminal output looks garbled, re-read the file with explicit UTF-8 before changing the text.
- Do not paste mojibake text into scripts, prefabs, markdown, or property tooltips. Replace corrupted Chinese with clean Chinese wording instead.
- Do not use runtime `findNodeByName`, `getChildByName`, or path lookup for new UI nodes unless matching unavoidable legacy behavior.
- Keep component logic focused on UI state and events. Put config/state helpers in `<Name>Data.ts`.
- When binding events manually, unbind/clear in lifecycle methods if item nodes are destroyed.

## Data Class Rules

Use `<Name>Data.ts` for:

- reading `window.app.dataProvider.getConfigData(...)`
- config normalization/fallbacks
- local save/load through `GetLocal`/`SetLocal`
- cost/reward/income calculations
- formatting helpers used by this module

Keep config key fallback names tolerant when table fields may change, for example `id || s_id || <module>_id`.

## Scroll List Pattern

When the module has a list:

Prefer the proven `MarketViewComp` / `Market_Win_fab` structure:

```text
ScrollView node       # cc.ScrollView, optional background Sprite
  view                # cc.Mask + cc.Graphics clipping node
    content           # UITransform, anchor usually (0.5, 1)
      def_item        # inactive item template
```

1. Prefab contains `ScrollView`, `view`, `content`, and inactive `def_item` template.
2. Put `cc.Mask` on the `view` node, not on the same node as `cc.ScrollView`.
3. Set `cc.ScrollView._content` to the `content` node.
4. Add `cc.Widget` to `view` and stretch it to match the parent `ScrollView` node size.
5. On `view`, set `cc.Mask._alphaThreshold` to `0.1` and keep the attached `cc.Graphics._color.a` at `255`; compare `OnlineReward_Win_fab` if unsure. This prevents active items under `content` from being clipped away by an invisible/over-strict mask.
6. Use top-anchored `content` and place items downward with negative Y, matching `OnlineRewardViewComp` / `MarketViewComp`.
7. `def_item` is mounted to ViewComp and has `<Name>ItemComp` mounted on it.
8. Clone with `instantiate(def_item)`, set parent to `content`, then call item component `setData(...)`.
9. Preserve scroll position on refreshes caused by item actions.
10. Only call `scrollToTop(0)` on first window show or explicit reset.
11. For small lists, build once and refresh item state in place; rebuild only when sorting/order changes.
12. For large lists, use a NodePool and visible-range refresh like `MarketViewComp`.
13. Avoid repeated icon/SpriteFrame loads during timed refresh; cache the current icon name in the item component.

Avoid:

- putting `Mask` and `ScrollView` on the same node with `content` as a direct child
- leaving `view`'s `cc.Graphics._color.a` as `0` or `cc.Mask._alphaThreshold` as `1`, which can make children visible outside `view` but invisible under `view/content`
- rebuilding the whole list every second with `destroy + instantiate`
- using code to permanently correct prefab base size, anchor, or hierarchy
- loading icon/SpriteFrame resources repeatedly during timer refreshes

Refresh pattern:

```ts
private refreshList(keepScroll = false): void {
    const scrollOffset = keepScroll ? this.scrollView?.getScrollOffset() : null;
    this.clearItems();
    // rebuild items...
    if (keepScroll && scrollOffset && this.scrollView) {
        this.scheduleOnce(() => this.scrollView?.scrollToOffset(scrollOffset, 0), 0);
    } else {
        this.scrollView?.scrollToTop(0);
    }
}
```

## Item Component Pattern

Item components should expose all visual resources through properties:

```ts
@ccclass("<Name>ItemComp")
export class <Name>ItemComp extends XComponent {
    @property({ type: Label, tooltip: "名称文本。" })
    public name_txt: Label | null = null;

    @property({ type: Node, tooltip: "未开启遮罩节点，资源由预制体挂载。" })
    public lock_mask_node: Node | null = null;

    public setData(data: I<Name>DisplayData, onClick: (data: I<Name>DisplayData) => void): void {
        // store data, bind click, refresh view
    }

    public Clear(): void {
        // unbind events and release references
    }
}
```

Common item states:

- unlocked/locked mask
- max level node
- action button text
- quality frame or rarity background
- portrait/icon Sprite from atlas
- cost icon and cost text

## Registration Checklist

Add or update these locations:

1. `WindowName.ts`
   - Add `WindowName.<Name>Win = "<Name>_Win_fab"`.
2. `ResDefine.ts`
   - Import `<Name>Win`.
   - Add `PrefabResDic[WindowName.<Name>Win] = { cls: <Name>Win, url: "prefabs/<module>view/<Name>_Win_fab", openKey: 0, resUrl: null }`.
   - Add atlas mapping only if this module uses a new atlas.
3. Entrance handler
   - Example: `this.app.CreateWindow(WindowName.<Name>Win)`.
   - When passing data, call `this.app.CreateWindow(WindowName.<Name>Win, null, data)`. Do not add an extra `null` before `data`; that slot only exists in `WindowManager.CreateWindow` for `openRegister`.
   - Stop event propagation when opening from clickable scene UI if needed.
4. `DataProvider.ts`
   - Add config JSON names to `tConfigBeforGame` only when the module has table configs.

## Prefab Checklist

Create under:

```text
assets/resources/prefabs/<module>view/<Name>_Win_fab.prefab
```

Prefab should include:

- root window node with the window prefab component pattern used by other windows
- UI layout designed for `720 x 1280`
- no `cc.Widget` on the root window node unless the user explicitly asks for root adaptive behavior
- `XButton` on all button nodes
- `XComponent` for custom UI components
- ViewComp mounted and all properties bound in Inspector/prefab JSON
- complete UI skeleton nodes required by the feature, including buttons, image placeholders, labels, panels, masks, list templates, effect placeholders, and other runtime-referenced nodes
- required Cocos components for the skeleton, such as `Sprite`, `Label`, `Widget`, `Mask`, `Layout`, `ScrollView`, and item components where needed
- close button node
- main panel nodes
- optional ScrollView/content/def_item
- optional item component mounted on `def_item`
- placeholder nodes for resources the user will bind later
- color-designed components set to `ffffffff` by default
- `cc.Widget` on nodes that need screen/parent-edge adaptation
- `FitSizeComp` or equivalent when device/resolution adaptation is needed
- Chinese property tooltips/comments for every exposed binding

Resource mounting rule: if the user says “组件类上不需要动态代码获取，全部通过挂载方式”, every referenced node/component must be exposed as a `@property` and bound on the prefab.

Concrete resources such as `SpriteFrame`, `SpriteAtlas`, and `AudioClip` may be left empty for the user to mount manually when the task does not explicitly require resource binding. The node, component, `@property` field, and prefab binding still must be generated. If the task explicitly asks to bind concrete resources, bind and verify them.

For ordinary module UI, nodes should come from the prefab. Do not dynamically create core buttons, item placeholders, backgrounds, pointers, or effect nodes in code unless the user explicitly requests generated UI.

## Config Table Workflow

When a window module needs config-backed data, use `.codex/skills/cocos-config-table/SKILL.md`.

Do not duplicate config-table rules here. The config skill owns workbook editing, export, `gameconfig.bin`, `DataProvider`, and verification rules.

## Verification

Before final response:

- Run `rg` checks for registration and config keys.
- Confirm every new window has `WindowBase` inheritance plus `showWindowAction`, `winDidShow`, `winWillHide`, and `winWillDestroy`.
- Confirm every new window is registered in both `WindowName.ts` and `ResDefine.ts`.
- Confirm window-opening call sites use the `App.CreateWindow` argument shape correctly: `CreateWindow(name, parent, data)` or `CreateWindow(name, parent, arg1, arg2)`, never `CreateWindow(name, parent, null, data)` unless the receiving `winDidShow` explicitly expects that leading null.
- Confirm prefab UI uses the `720 x 1280` design baseline, root node has no default `cc.Widget`, color-designed components default to `ffffffff`, and required non-root `cc.Widget` components exist.
- Confirm buttons use `XButton`, new custom components use `XComponent`, and color-designed components default to `ffffffff`.
- Confirm required feature nodes and components are generated even if concrete art/audio resources are left empty for manual Inspector binding.
- Report every `SpriteFrame`, `SpriteAtlas`, `AudioClip`, atlas image, or other concrete resource that the user still needs to mount manually.
- If needed tools such as Node/npm, TypeScript, Cocos Creator, Python, Excel dependencies, or PowerShell script permission are unavailable, report the missing environment and the blocked verification step instead of silently skipping it.
- For every `ScrollView`, confirm the `view` node has `cc.Widget` stretched to the parent size.
- For every `ScrollView`, confirm the `view` clipping node uses a working mask setup: `cc.Mask._alphaThreshold = 0.1` and `cc.Graphics._color.a = 255`, unless an existing nearby prefab intentionally uses different values.
- Confirm exposed `@property` fields have Chinese tooltips/comments and no garbled Chinese text.
- When config tables are touched, run the verification checklist in `.codex/skills/cocos-config-table/SKILL.md`.
- When checking Chinese text in PowerShell, use `Get-Content -Encoding UTF8` or another explicit UTF-8 read before judging whether content is corrupted.
- Parse generated prefab JSON when it was edited mechanically.
- Run `tsc.cmd -p tsconfig.json --noEmit --ignoreDeprecations 6.0` if available.
- If `tsc` fails due existing Cocos declaration or legacy module errors, report that clearly and note whether new files appear in the error list.
- Report any resource nodes the user still needs to bind.

## Common Mistakes

- Forgetting `PrefabResDic`, so `CreateWindow` cannot open the prefab.
- Forgetting `WindowName`, so entrance handler has no stable key.
- Passing an extra `null` through `this.app.CreateWindow(...)` before real data because `WindowManager.CreateWindow` has an `openRegister` parameter; this shifts config from `arg[0]` to `arg[1]` and makes `winDidShow(config)` receive `null`.
- Leaving `ScrollView._content` null, causing the list not to scroll.
- Making `ScrollView/view`'s mask graphics transparent or using an alpha threshold of `1`; children may disappear only when placed under `view/content`, while appearing normal if dragged outside the clipped view.
- Calling `scrollToTop(0)` after every item action, which loses the user's list position.
- Using dynamic node search in new components when the module should be Inspector-mounted.
- Exposing `@property` fields without Chinese tooltips/comments, making prefab binding unclear in Inspector.
- Trusting garbled terminal output and then saving mojibake back into Chinese comments or tooltips.
- Running the export script from the wrong working directory, which can produce files in the wrong `assets/` path.
- Adding config JSON to `DataProvider.ts` but not rebuilding `gameconfig.bin`.
