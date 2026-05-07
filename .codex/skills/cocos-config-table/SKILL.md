---
name: cocos-config-table
description: Use when editing, adding, exporting, or validating Cocos config tables, including 游戏配置表.xls sheets, _out_config JSON, gameconfig.bin, DataProvider registration, JSZip loading, and config-backed modules.
---

# Cocos Config Table

## Scope

Use this skill whenever a task touches configuration tables, generated config JSON, `gameconfig.bin`, `DataProvider` registration, or a module that reads table-driven data. For project-specific sheet names and module consumers, also read the project-level config or module skill when present.

The table source of truth is the `.xls` file. Generated JSON and `gameconfig.bin` are export artifacts; do not treat them as the only change.

## Key Paths

- Source workbook: `cocos_game_testing/config/游戏配置表.xls`
- Generated JSON: `cocos_game_testing/config/_out_config_/*.json`
- Packed runtime config: `cocos_game_testing/assets/resources/assets/config/gameconfig.bin`
- Runtime registration: `cocos_game_testing/assets/script/funsan/core/util/DataProvider.ts`
- Runtime load path: `resources.load("assets/config/gameconfig", BufferAsset, ...)`, without `.bin`
- Runtime helpers: `cocos_game_testing/assets/script/funsan/core/util/ConfigHelper.ts`

If a local export tool is present, it may be named `视频打包工具.exe` or wrap `xls转json和confog转bin导表批处理.py`. When running the Python exporter directly, run it from its expected `tool/` directory; running from the repo root can shift output paths.

## Environment Checks

Before editing or exporting config tables, check only the tools required by the task:

- Python availability when running export or workbook scripts.
- Legacy Excel packages such as `xlrd`, `xlwt`, and `xlutils` when editing `.xls` by script.
- Export tool path and expected working directory.
- Node/npm/TypeScript only when runtime code verification is part of the task.
- PowerShell execution policy when invoking `.ps1` helpers.

If a required tool is missing, report what is missing, which workflow is blocked, and what install/config step is needed. Do not silently skip export or verification. If installation needs network access, global changes, or elevated permissions, ask first.

## Workbook Rules

1. Add or change config data in `游戏配置表.xls` first.
2. Keep Chinese text readable. PowerShell can display mojibake; verify with explicit UTF-8 reads or the editor before saving text back.
3. For new or adjusted sheets, use `游戏基础配置` A1:F5 as the fixed style baseline: font, font size, font color, fill color, alignment, borders, number format, row height, and A-F column widths.
4. Row 5 is the exported field-name row and must be black background with white text. If `游戏基础配置` or a copied source sheet is inconsistent, force row 5 to black fill plus white font before saving.
5. F column should follow E column width/style when extending the fixed header area.
6. Business columns after A-F should use a consistent wide width, roughly 100 px where the tool can express it.
7. Header labels, output file names, and field names may change for business needs, but the fixed header style should not drift.
8. Existing manually styled sheets must keep their own style. In particular, `手机聊天角色配置` and `手机聊天内容配置` are style sources for future phone-chat edits.
9. When adding columns to phone-chat sheets, copy adjacent business-column width, font, fill, alignment, borders, and number format. Do not overwrite those sheets with a generic script style.

## Sheet Design

- Keep fields planner-friendly and stable.
- Any new sheet/sub-table field that stores coded values, enum-like strings, numeric types, workflow keys, category/type ids, mode switches, condition keys, reward keys, jump targets, or asset keys must explain its allowed values for planners in the sheet's Chinese description row or a nearby note. This is a general rule for all typed config fields, not a whitelist of specific columns.
- Prefer `value=meaning` wording in the planner-facing row. Examples only: `reward_type: gold=金币；click_income_percent=点击收益百分比加成`, `go_to: click=主界面点击；secretary=秘书；bank=钱庄`, `progress_mode: 1=历史累计；2=从当前任务出现后开始累计`.
- If a coded field is used in TypeScript, update the related interface or parser comments together with the workbook so the `.xls`, exported JSON, and runtime code stay aligned.
- Use an `off` field when the existing config pattern supports disabling rows.
- Use stable ids/keys if code will cache the sheet as a dictionary.
- Keep source names aligned: workbook sheet -> exported JSON name -> `DataProvider` key -> code lookup.
- For phone-chat trigger conditions, use one `condition` string such as `level:1` or `gold:100|props_102:1`; do not split it back into `condition_key`, `condition_op`, `condition_value`.
- For prize-wheel-like rows, the preferred fields are `off`, `slot`, `multiplier`, `icon`, `weight`, `duration_sec`, `spin_time`, `desc`.

## Editing `.xls`

Old `.xls` files need legacy Python packages when edited by script:

```powershell
python -m pip install --target .pip_lib --cache-dir .pip_tmp xlrd==1.2.0 xlwt==1.3.0 xlutils==2.0.0
```

Use `xlrd.open_workbook(..., formatting_info=True)` plus `xlutils.copy` when preserving styles. Prefer copying styles from existing cells rather than recreating style objects by guesswork.

After export or scripted editing, remove temporary `.pip_lib` and `.pip_tmp` when they were created only for the task.

## Export Workflow

1. Save the workbook.
2. Run the project export tool from its expected working directory.
3. Confirm the changed JSON exists in `cocos_game_testing/config/_out_config_/`.
4. Confirm `cocos_game_testing/assets/resources/assets/config/gameconfig.bin` was rebuilt.
5. Inspect `gameconfig.bin` as a zip and confirm it contains the JSON that runtime will load.

Example inspection:

```powershell
python -c "import json, zipfile; z=zipfile.ZipFile(r'cocos_game_testing/assets/resources/assets/config/gameconfig.bin'); print(z.namelist())"
```

For a specific file:

```powershell
python -c "import json, zipfile; z=zipfile.ZipFile(r'cocos_game_testing/assets/resources/assets/config/gameconfig.bin'); print(json.loads(z.read('challengeConfig.json').decode('utf-8'))[:1])"
```

## Runtime Registration

`DataProvider.tConfigBeforGame` must only list JSON files actually present in `gameconfig.bin`.

Each entry shape is:

```ts
["fileName.json", "cacheKey"]
["fileName.json", "cacheKey", ["idField"]]
["fileName.json", "cacheKey", ["groupField"], true]
```

Do not add a JSON file to `DataProvider` until the exported `gameconfig.bin` contains it. Missing files should log and continue, not block Loading.

The loader must remain compatible with both `JSZip.loadAsync` and `JSZip.default.loadAsync`, because Cocos preview/build wrapping can differ.

`ConfigHelper` should tolerate initialization order by re-reading `this.app.dataProvider` when needed instead of relying only on the constructor cache.

## Verification Checklist

Before finishing a config-table task:

- Re-open the `.xls` or read it with `xlrd` to confirm the intended rows, fields, and styles were saved.
- Confirm row 5, the exported field-name row, is black background with white text across all used columns.
- Confirm coded/config type fields have planner-facing value explanations in the workbook, especially enum, numeric mode, string switch, reward, jump, and condition fields.
- Check the generated JSON in `cocos_game_testing/config/_out_config_/`.
- Inspect `gameconfig.bin` and confirm the JSON file is inside it.
- If `DataProvider.ts` changed, confirm every registered JSON exists in the bin.
- Search config keys and file names with `rg` so code lookups match the exported names.
- For runtime loader changes, confirm there are no console errors like `JSZip.loadAsync is not a function`, `Cannot read properties of null (reading 'async')`, or `Cannot read properties of null (reading 'getConfigData')`.
- TypeScript checks may show historical Cocos or `SceneManager` noise; distinguish new errors from existing ones.

## Common Mistakes

- Editing generated JSON but not the workbook.
- Adding bare enum/code columns without explaining allowed values for planners.
- Rebuilding JSON but forgetting `gameconfig.bin`.
- Registering a config in `DataProvider.ts` before it exists in the bin.
- Running the exporter from the wrong working directory.
- Overwriting manually styled sheets with default script styles.
- Trusting garbled terminal output and saving corrupted Chinese text.
- Forgetting that `resources.load` uses `assets/config/gameconfig`, not `assets/config/gameconfig.bin`.
- Leaving temporary dependency folders in the project after a one-off export.
