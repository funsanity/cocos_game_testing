---
name: cocos-prefab-authoring
description: Use when creating, editing, reviewing, or validating Cocos Creator prefab UI, node bindings, buttons, colors, Widgets, FitSizeComp adaptation, Inspector properties, and mounted component conventions.
---

# Cocos Prefab Authoring

## Baseline

Use this skill for prefab and UI authoring rules that apply across Cocos projects using this Harness. For window registration, also use `cocos-window-module`. For project-specific prefab families, read the project skill too.

## Required UI Rules

- Design new or modified UI for `720 x 1280`.
- Ordinary window root nodes stay `720 x 1280`.
- Do not add `cc.Widget` to the root window node by default.
- Use `cc.Widget` for internal nodes that must adapt to parent edges or screen size.
- Use `FitSizeComp` or the project equivalent for device/resolution adaptation.
- All components or elements with color design default to `ffffffff`.
- Change color only for explicit states: selected, disabled, warning, mask, guide, transition, or effect.

## Component And Binding Rules

- Buttons in prefabs must mount/use `XButton`.
- New custom UI scripts must extend/use `XComponent`.
- Expose important UI nodes and components as `@property` fields.
- Every public property or Inspector field must have a Chinese comment or tooltip.
- Prefer mounted references over `getChildByName`, path lookup, and runtime node discovery.
- If a node is required for the feature to work, it must be bound in the prefab or reported as a required binding.

## Node/Component Generation Rules

- AI/tools must generate the stable prefab node structure and required component mounts for the requested feature.
- Required feature nodes include buttons, image placeholders, labels, list templates, backgrounds, masks, pointers, effects, panels, and other placeholder nodes the runtime/editor logic references.
- Required components must be present even when final art resources are not bound yet: `XButton` for buttons, `XComponent`-derived scripts for custom UI, and `Widget`, `Mask`, `Layout`, `ScrollView`, `Sprite`, `Label`, or other Cocos components needed by the structure.
- Expose and bind all required node/component references through `@property` fields. Do not leave runtime code to discover required nodes by name later.
- `SpriteFrame`, `SpriteAtlas`, `AudioClip`, and similar concrete resources may be left for the user to mount manually in Cocos Inspector when the task does not explicitly require automatic resource binding.
- Even when resources are left empty, the placeholder node, component, `@property` field, and prefab binding must exist and be stable.
- If the task explicitly asks to bind concrete resources, bind them and verify the paths/references instead of leaving them for manual mounting.

## Prefab Editing Policy

- Do not hand-edit `.scene` or `.meta` files unless the task explicitly requires asset-file edits and the serialized structure is understood.
- Do not change `.prefab` files casually when the task does not ask for UI/prefab/node/component work.
- If the user explicitly asks for prefab, UI structure, nodes, Layout, components, bindings, or mounted placeholders, update the `.prefab` and follow this skill.
- Prefer using existing working prefabs as structural references.
- Do not dynamically create permanent UI structure in code when the prefab should own it.
- Keep images, labels, buttons, placeholders, list templates, and effect nodes in the prefab unless the user asks for generated UI.

## ScrollView Rules

- Use this structure for ordinary lists:

```text
ScrollView
  view
    content
      def_item
```

- `view` owns clipping with `cc.Mask`.
- `cc.ScrollView._content` points to `content`.
- Add `cc.Widget` to `view` when it must stretch to the ScrollView parent.
- Use inactive `def_item` as the template and mount item component properties in the prefab.

## Validation

- Confirm default colors are `ffffffff` unless state-specific.
- Confirm `XButton` and `XComponent` usage on new UI.
- Confirm public properties have Chinese tooltips/comments.
- Confirm required bindings are present or clearly reported.
- Confirm required nodes/components exist even if concrete resources are left for the user to mount manually.
- Report every `SpriteFrame`, `SpriteAtlas`, `AudioClip`, or other resource that still needs manual Inspector binding.
- If editing serialized prefab JSON, parse or search it after changes to verify node/component references.

## Common Mistakes

- Treating `cc.Sprite` as the only color rule; the default applies to every color-designed component.
- Forgetting `XButton` on clickable prefab nodes.
- Adding root `cc.Widget` because inner adaptive nodes were missing.
- Leaving important UI nodes discoverable only by name at runtime.
