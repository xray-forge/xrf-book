# Gamedata CLI

Gamedata commands verify assembled assets and show which files a game installation resolves. Run verification after a
build or asset import, before launching or packaging the result.

## `gamedata verify`

From a project containing `target/gamedata`:

```powershell
xrf-cli gamedata verify --path ./target/gamedata --report ./verification-report.json
```

Example output excerpt — verify textures:

```text
Verify textures:
Verified gamedata textures in 6 ms, 3/3 textures valid; 0/0 declared bumps resolved
Gamedata project checked in 9 ms
Gamedata project: passed
```

Exit code: `0`.

`-p, --path` must name an existing gamedata directory or an installation declaring its mounted sources. The resolved
tree must contain `configs/system.ltx`; it may come from a loose file or an archive.

All asset checks run unless `--checks` selects a subset. Add `--strict` to fully validate expensive payloads and apply
the checks' stricter content requirements:

```powershell
xrf-cli gamedata verify --path ./target/gamedata --checks scripts,ltx
xrf-cli gamedata verify --path ./target/gamedata --checks sounds --strict
```

A check that finds something prints one failure line naming the check, its status, and its finding count. Add
`--verbose` to print each finding beneath it as `[rule] subject: message`, or read the findings from the
[report](#json-report).

`--jobs` controls parallel work; `-j 1` makes execution sequential. Worker count should not change the findings for
unchanged input, but durations, cache statistics, and execution metadata can differ.

By default the run ignores `.git`, `.idea`, `particles_unpacked`, `textures_unpacked`, `.gitignore`, `.gitattributes`,
`README.md`, and `LICENSE`. Supplying `--ignore` replaces this list; include every exclusion the run still needs. Repeat
the flag or list several entries after one, separated by spaces:

```powershell
xrf-cli gamedata verify --path ./target/gamedata --ignore .git textures_unpacked docs
```

## Patched installations

For Anomaly and Monolith-based installations, enable the [DLTX dialect](ltx.md#the-dltx-patch-dialect):

```powershell
xrf-cli gamedata verify --path "C:/Games/Anomaly" --dltx --report ./anomaly-report.json
```

Checks then resolve patched config values. Leave `--dltx` off for standard vanilla or OpenXRay LTX trees.

Anomaly's Monolith engine also reads some configs differently from OpenXRay. Its weather keyframes carry no sun angles,
for example, because it positions the sun from a table instead. By default, `--engine auto` detects the engine from the
tree: Anomaly's launcher or renderer executables, its `$warfare_presets$` mount in `fsgame.ltx`, or its
`[weather_cycles]` graph in `configs/environment/dynamic_weather_graphs.ltx` mark it `extended`; anything else reads as
`vanilla` (OpenXRay and Call of Pripyat). The report names the evidence. Name the engine explicitly when the tree shows
none of these:

```powershell
xrf-cli gamedata verify --path "C:/Games/Anomaly" --dltx --engine extended --report ./anomaly-report.json
```

The two options are independent: `--dltx` picks how configs are resolved, and `--engine` picks how the resolved values
are read.

## Inspect resolved assets

Use `gamedata list` to locate a winning file and inspect files hidden by higher-priority mounts:

```powershell
xrf-cli gamedata list --path "C:/Games/Anomaly" --prefix textures --shadowed `
  --report ./asset-list.json
```

Example output — list textures:

```text
Listing ./gamedata
  directory ./gamedata (gamedata)
  textures\act_cat_bump.thm [./gamedata]
  textures\prop_lampa_g.dds [./gamedata]
  textures\ui\ui_test_sheet.dds [./gamedata]
  textures\ui_empty.dds [./gamedata]
4 asset(s) across 1 mount(s) in 1 ms
```

Exit code: `0`.

The default source mode searches for a containing installation. Use `--source directory` to inspect only a loose tree,
or `--loose` to omit archived entries. Repeat `--path` to layer roots, highest priority first.

Each mount line names the mount's kind, `directory` or `archive`, then its root and label. The report's `mounts` lists
the same mounts in search order as `{ kind, label, root }` objects, where `root` is the directory, or the directory
holding the archive volumes.

Console output shows at most 40 entries per section; the report retains the full list. Review skipped-source warnings as
well as the winning paths: a successfully produced listing may still be incomplete.

## Checks and rules

Select checks by their group names: `animations`, `environment`, `levels`, `ltx`, `materials`, `meshes`, `particles`,
`particles-usage`, `scripts`, `shaders`, `sounds`, `spawns`, `textures`, and `weapons`.

Findings carry a stable rule identifier. These groups expose the following rules:

- **Animations:** `animations.hud-item`, `animations.motion-collision`, `animations.player-hud`,
  `animations.object-motion-read`, `animations.object-motion-round-trip`, `animations.object-motion-value`,
  `animations.post-process-read`, `animations.post-process-round-trip`, `animations.post-process-value`,
  `animations.post-process-noise-granularity`. Besides the HUD animations a weapon or device names, this check reads
  every object motion (`.anm`, `.anm1`) and post-process effect (`.ppe`) the game holds.
  - `animations.player-hud`: a problem with the player hands. Each hands section gets one finding when its visual is
    missing, its motion refs or a linked motion bank does not read, or it plays no animations. A motion a weapon's HUD
    section plays is found as the engine finds it, under its own name or as one of the numbered variants `name1` to
    `name8`. One that some hands lack is one finding, naming the HUD sections and fields that ask for it, the hands that
    lack it, and what the engine then does: the vanilla engine asserts, and the extended engine plays `hand_idle_doun`
    in its place where the hands have it. A motion ref that resolves to no bank is reported by the meshes check.
  - `*-read`: the file does not read, for example because it is truncated or declares a version the engine does not
    load.
  - `*-round-trip`: the file does not write back to the bytes it was read from. An object motion that carries a chunk
    besides its motion is the usual case: the engine skips that chunk, and the tools do not keep it.
  - `*-value`: a channel evaluates to something other than a finite number, at a key, between two keys, or past them
    where the envelope's behaviour takes over. The finding names the channel and the time, in seconds.
  - `animations.post-process-noise-granularity`: an effect's keyed noise granularity reaches zero or below at a time the
    effect plays, from its start to its length. The engine stops with an assertion once the effect plays at full
    strength there. The finding names the granularity and the time, in seconds.
- **Environment:** `environment.engine`, `environment.reference`, `environment.convention`, `environment.asset`. This
  check covers everything under `configs\environment` that the engine reads: weather cycles and effects, suns,
  thunderbolts, ambients with their sound channels and effects, and Anomaly's sun table.
  - `environment.engine`: something the engine refuses to load, or reads as something other than what is written, such
    as a short vector read with zeros.
  - `environment.reference`: a section names another section the engine cannot find.
  - `environment.convention`: something the engine loads but warns about or silently corrects, such as a colour outside
    its checked range, a clamped value, or text after a number. Problems in a definition that nothing references are
    reported under this rule too, because the engine never loads that definition.
  - `environment.asset`: a sky or clouds texture that the game does not contain.
- **Levels:** `levels.ai-guid`, `levels.ai-node-count`, `levels.ai-version`, `levels.cform-version`,
  `levels.details-pair`, `levels.file-empty`, `levels.file-truncated`, `levels.game-material`, `levels.graph-duplicate`,
  `levels.graph-guid`, `levels.header-version`, `levels.level-guid`, `levels.ltx-read`, `levels.map-texture`,
  `levels.missing-bundle`, `levels.missing-file`, `levels.orphan-bundle`, `levels.roster-conflict`,
  `levels.shader-reference`, `levels.shaders-chunk`, `levels.texture-reference`, `levels.undeclared-map`.
  `levels.game-material`: collision faces made of a material `gamemtl.xr` does not define, or of a dynamic one; either
  stops the level from loading.
- **LTX:** `ltx.formatting`, `ltx.schema`, `ltx.verification`.
- **Materials:** `materials.engine`, `materials.library`, `materials.list`, `materials.mark`, `materials.pair`,
  `materials.reference`. This check reads the game material library, `gamemtl.xr`.
  - `materials.library`: the library is missing or does not read. The engine then loads no materials, and the other
    checks below cannot run.
  - `materials.engine`: a material the engine looks up by a name of its own (`default`, `default_object`,
    `objects\small_box`, `objects\bullet`, `objects\knife`) is missing, or `default_object` is not dynamic.
  - `materials.pair`: a pair names a material number the library does not define.
  - `materials.list`: a pair lists more than six sounds, or more than four particles or marks, in one list. The engine
    stops with an assertion while loading the library.
  - `materials.mark`: a wallmark texture a pair names is missing. The finding counts the pairs that name it.
  - `materials.reference`: a `material` key in a config, or a climable object in a spawn file, names a material the
    library does not define. Names match regardless of ASCII case, as in the engine.
- **Meshes:** `meshes.chunk-residue`, `meshes.game-material`, `meshes.motion-label`, `meshes.motion-read`,
  `meshes.motion-validation`, `meshes.read`, `meshes.shader-library`, `meshes.validation`. `meshes.game-material`: a
  skeleton bone made of a material the library does not define, or of a static one, which the engine asserts against.
- **Particles:** `particles.library`, `particles.texture`.
- **Particle usage:** `particles-usage.reference`, `particles-usage.spawn`, `particles-usage.spawn-custom-data`. The
  collide particles of material pairs count as references, reported once per name against `gamemtl.xr`.
- **Scripts:** `scripts.read`, `scripts.syntax`. `scripts.read`: the script does not read as Windows-1251 text.
  `scripts.syntax`: LuaJIT's parser rejects the script, and the finding gives the parser's message.
- **Shaders:** `shaders.include-cycle`, `shaders.include-missing`, `shaders.include-syntax`, `shaders.lua-syntax`,
  `shaders.renderer-root`, `shaders.source-invalid`, `shaders.source-read`.
- **Sounds:** `sounds.files`, `sounds.references`. The breaking, step and collide sounds of material pairs count as
  references, reported once per name against `gamemtl.xr`.
- **Spawns:** `spawns.read`.
- **Textures:** `textures.bump`, `textures.bump-companion`, `textures.bump-declaration`, `textures.chunk-residue`,
  `textures.read`, `textures.dds`.
- **Weapons:** `weapons.validation`, one finding per problem with a weapon section, each naming the section: a visual,
  HUD section, item visual or motion bank that is missing or does not read, a required sound that is not declared, a
  sound that does not resolve, a layered sound section that breaks its numbering or takes a volume or delay, and a
  launcher-capable weapon missing the launcher bores its `anm_bore` needs.

Without a readable `gamemtl.xr`, only `materials` fails; the levels, meshes, particle usage and sounds checks leave out
what they would resolve against the library.

Two checks always run and cannot be selected or suppressed with `--checks`: `collisions.unreachable` reports files no
lookup can reach, and `coverage.skipped-mount` reports declared sources that could not be opened. `checks.execution`
identifies a check that failed to execute.

A collision is two files of one source folding to a single engine identity, where nothing ranks them and one simply
cannot be read. It is not the same as an override: a patch volume or a loose `gamedata` file claiming a path a
lower-priority source also holds is the arrangement working as intended, has a defined winner, and never fails a
verdict. Use `gamedata list --shadowed` to see those.

### Interpret common findings

Animation validation allows missing item motions where the engine falls back to `idle`. Duplicate motion names across
banks in one HUD namespace are reported because lookup is ambiguous.

For `meshes.chunk-residue`, inspect the model and use [`ogf fix`](ogf.md#ogf-fix) for recognized unread tails.

For textures, distinguish four repairs:

| Rule                        | Meaning and action                                                                                      |
| --------------------------- | ------------------------------------------------------------------------------------------------------- |
| `textures.bump`             | A declared bump does not resolve. Restore the file, repoint the descriptor, or disable the declaration. |
| `textures.bump-companion`   | The bump exists but its `#` companion is missing. Restore or generate the pair.                         |
| `textures.bump-declaration` | The descriptor carries a declaration the engine does not use. Correct the descriptor.                   |
| `textures.chunk-residue`    | A descriptor chunk is longer than its fields. The engine ignores the bytes; resave the descriptor.      |

Companion, unused-declaration and chunk-residue findings are reported in normal mode but fail verification only under
`--strict`. See [THM bump declarations](thm.md) for descriptor edits and
[DDS bump generation](dds.md#generate-a-bump-pair) for missing texture pairs.

## JSON report

The saved file uses the [shared report envelope](cli.md#reporting). Its `result` contains overall `status`, `checks`,
`duration`, `engine`, `cache`, and `skippedMounts`. `reads` is included only with `--trace-reads`. `engine` names the
engine the configs were read as and the evidence that decided it.

Each check is `{ id, status, duration, findings, summary }`. `id` is the check's group name, such as `textures`, and
`summary` is the check's one-line verdict. Each finding is `{ ruleId, subject, message }`.

Example report finding — inspect a missing mesh dependency:

```json
{
  "message": "Mesh references missing motion 'dynamics\\devices\\dev_bolt\\dev_bolt_hud_animation'",
  "ruleId": "meshes.motion-validation",
  "subject": "meshes/ogf/dev_bolt_hud.ogf"
}
```

Exit code: `3`. The referenced animation bank is missing.

`subject` is relative to the verified root when the asset lies beneath it, and `null` when the finding has no asset
subject. `message` is human-readable; use `ruleId` for automated classification. Findings are ordered by subject, rule,
and message.

Statuses are `passed`, `failed`, `error`, `incomplete`, or `skipped`; the overall status reflects the most severe check
result. A check's duration is `null` when it did not run; measured durations are whole milliseconds. `skippedMounts`
identifies declared sources omitted because they could not be opened.

### Inspect cache and read costs

`cache` reports retained entries and bytes, hits, misses, and refusals. Hits plus misses count parsed-asset requests,
including misses for asset kinds the cache does not retain. A non-zero `refused` count means the byte ceiling prevented
retention.

Add read tracing when investigating repeated I/O:

```powershell
xrf-cli gamedata verify --path ./target/gamedata --trace-reads --report ./read-report.json
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
