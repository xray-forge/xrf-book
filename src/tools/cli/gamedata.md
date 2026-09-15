# Gamedata CLI

Gamedata commands verify assembled assets and show which files a game installation resolves. Run verification after a
build or asset import, before launching or packaging the result.

## `gamedata verify`

From a project containing `target/gamedata`:

```powershell
xrf-cli gamedata verify ./target/gamedata --report ./verification-report.json
```

Example output excerpt — verify textures:

```text
Verify textures:
Verified gamedata textures in 6 ms, 3/3 textures valid; 0/0 declared bumps resolved

Project gamedata is valid
Gamedata project verified in 9 ms
```

Exit code: `0`.

The positional root must be an existing gamedata directory or an installation declaring its mounted sources. The
resolved tree must contain `configs/system.ltx`; it may come from a loose file or an archive.

All asset checks run unless `--checks` selects a subset. Add `--strict` to fully validate expensive payloads and apply
the checks' stricter content requirements:

```powershell
xrf-cli gamedata verify ./target/gamedata --checks scripts,ltx
xrf-cli gamedata verify ./target/gamedata --checks sounds --strict
```

`--jobs` controls parallel work; `-j 1` makes execution sequential. Worker count should not change the findings for
unchanged input, but durations, cache statistics, and execution metadata can differ.

By default the run ignores `.git`, `.idea`, `particles_unpacked`, `textures_unpacked`, `.gitignore`, `.gitattributes`,
`README.md`, and `LICENSE`. Supplying `--ignore` replaces this list; include every exclusion the run still needs.

## Patched installations

For Anomaly and Monolith-based installations, enable the [DLTX dialect](ltx.md#the-dltx-patch-dialect):

```powershell
xrf-cli gamedata verify "C:/Games/Anomaly" --dltx --report ./anomaly-report.json
```

Checks then resolve patched config values. Leave `--dltx` off for standard vanilla or OpenXRay LTX trees.

## Inspect resolved assets

Use `gamedata list` to locate a winning file and inspect files hidden by higher-priority mounts:

```powershell
xrf-cli gamedata list --path "C:/Games/Anomaly" --prefix textures --shadowed `
  --report ./asset-list.json
```

Example output — list textures:

```text
Listing ./gamedata
  Directory ./gamedata (gamedata)
  textures\act_cat_bump.thm [./gamedata]
  textures\prop_lampa_g.dds [./gamedata]
  textures\ui\ui_test_sheet.dds [./gamedata]
  textures\ui_empty.dds [./gamedata]
4 asset(s) across 1 mount(s) in 1 ms
```

Exit code: `0`.

The default source mode searches for a containing installation. Use `--source directory` to inspect only a loose tree,
or `--loose` to omit archived entries. Repeat `--path` to layer roots, highest priority first.

Console output shows at most 40 entries per section; the report retains the full list. Review skipped-source warnings as
well as the winning paths: a successfully produced listing may still be incomplete.

## Checks and rules

Select checks by their group names: `animations`, `levels`, `ltx`, `meshes`, `particles`, `particles-usage`, `scripts`,
`shaders`, `sounds`, `spawns`, `textures`, `weapons`, and `weathers`.

Findings carry a stable rule identifier. These groups expose the following rules:

- **Animations:** `animations.hud-item`, `animations.motion-collision`, `animations.player-hud`.
- **Levels:** `levels.ai-guid`, `levels.ai-node-count`, `levels.ai-version`, `levels.cform-version`,
  `levels.details-pair`, `levels.file-empty`, `levels.file-truncated`, `levels.graph-duplicate`, `levels.graph-guid`,
  `levels.header-version`, `levels.level-guid`, `levels.ltx-read`, `levels.map-texture`, `levels.missing-bundle`,
  `levels.missing-file`, `levels.orphan-bundle`, `levels.roster-conflict`, `levels.shader-reference`,
  `levels.shaders-chunk`, `levels.texture-reference`, `levels.undeclared-map`.
- **LTX:** `ltx.formatting`, `ltx.schema`, `ltx.verification`.
- **Meshes:** `meshes.chunk-residue`, `meshes.motion-label`, `meshes.motion-read`, `meshes.motion-validation`,
  `meshes.path`, `meshes.read`, `meshes.shader-library`, `meshes.validation`.
- **Particles:** `particles.library`, `particles.texture`.
- **Particle usage:** `particles-usage.reference`, `particles-usage.spawn`, `particles-usage.spawn-custom-data`.
- **Scripts:** `scripts.path`, `scripts.read`, `scripts.syntax`. Syntax checks use the LuaJIT dialect.
- **Shaders:** `shaders.include-cycle`, `shaders.include-missing`, `shaders.include-syntax`, `shaders.lua-syntax`,
  `shaders.renderer-root`, `shaders.source-invalid`, `shaders.source-read`.
- **Sounds:** `sounds.files`, `sounds.references`.
- **Spawns:** `spawns.path`, `spawns.read`.
- **Textures:** `textures.bump`, `textures.bump-companion`, `textures.bump-declaration`, `textures.path`,
  `textures.read`, `textures.dds`.
- **Weapons:** `weapons.validation`.
- **Weathers:** `weathers.definitions`, `weathers.files`, `weathers.validation`.

Two checks always run and cannot be selected or suppressed with `--checks`: `collisions.unreachable` reports logical
path collisions, and `coverage.skipped-mount` reports declared sources that could not be opened. `checks.execution`
identifies a check that failed to execute.

### Interpret common findings

Animation validation allows missing item motions where the engine falls back to `idle`. Duplicate motion names across
banks in one HUD namespace are reported because lookup is ambiguous.

For `meshes.chunk-residue`, inspect the model and use [`ogf fix`](ogf.md#ogf-fix) for recognized unread tails.

For textures, distinguish three repairs:

| Rule                        | Meaning and action                                                                                      |
| --------------------------- | ------------------------------------------------------------------------------------------------------- |
| `textures.bump`             | A declared bump does not resolve. Restore the file, repoint the descriptor, or disable the declaration. |
| `textures.bump-companion`   | The bump exists but its `#` companion is missing. Restore or generate the pair.                         |
| `textures.bump-declaration` | The descriptor carries a declaration the engine does not use. Correct the descriptor.                   |

Companion and unused-declaration findings are reported in normal mode but fail verification only under `--strict`. See
[THM bump declarations](thm.md) for descriptor edits and [DDS bump generation](dds.md#generate-a-bump-pair) for missing
texture pairs.

## JSON report

The saved file uses the [shared report envelope](cli.md#reporting). Its `result` contains `checks`, overall `status`,
`duration`, `cache`, and `skippedMounts`. `reads` is included only with `--trace-reads`.

Each check has its own status, duration, summary, verification type, and findings.

Example report finding — inspect a missing mesh dependency:

```json
{
  "assetPath": "meshes/ogf/dev_bolt_hud.ogf",
  "message": "Mesh references missing motion 'dynamics\\devices\\dev_bolt\\dev_bolt_hud_animation'",
  "ruleId": "meshes.motion-validation"
}
```

Exit code: `3`. The referenced animation bank is missing.

`assetPath` is root-relative when available and `null` when the finding has no asset subject. `message` is
human-readable; use `ruleId` for automated classification. Findings are ordered by asset path, rule, and message.

Statuses are `passed`, `failed`, `error`, `incomplete`, or `skipped`; the overall status reflects the most severe check
result. A check's duration is `null` when it did not run; measured durations are whole milliseconds. `skippedMounts`
identifies declared sources omitted because they could not be opened.

### Inspect cache and read costs

`cache` reports retained entries and bytes, hits, misses, and refusals. Hits plus misses count parsed-asset requests,
including misses for asset kinds the cache does not retain. A non-zero `refused` count means the byte ceiling prevented
retention.

Add read tracing when investigating repeated I/O:

```powershell
xrf-cli gamedata verify ./target/gamedata --trace-reads --report ./read-report.json
```

`reads` reports `paths`, `reads`, `bytes`, `uniqueBytes`, and the 25 hottest paths. The difference between total and
unique bytes exposes repeated reads. The path count covers the whole run even though the hottest-path list is capped.
Tracing adds synchronization on the read path; compare timings with the same tracing setting.

## Result

A fully passed result exits 0. Invalid content exits 3; error, incomplete, and skipped results exit 1. In particular,
missing mounted sources cannot produce a clean verification verdict merely because the remaining assets passed.

Verification covers the resolved assets in the selected scope, including generated scripts and configs. It does not
validate source files omitted from the build or replace testing the resulting game behavior.

## Command reference

{{#include reference/gamedata.md:commands}}
