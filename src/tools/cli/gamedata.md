# Gamedata CLI

Gamedata commands validate one assembled gamedata directory. Run them after building gamedata and before launching or
packaging it.

## `gamedata verify`

```powershell
xrf-cli gamedata verify ./target/gamedata
```

`ROOT` is the required positional path to the assembled gamedata directory. The command reads configs from
`ROOT/configs` and requires `ROOT/configs/system.ltx`.

If `--checks` is omitted, all checks run. `--strict` fully validates expensive asset payloads; it is long-only, because
`-s` means `--silent` on every command.

Accepted check names are `animations`, `levels`, `ltx`, `meshes`, `particles`, `particles-usage`, `scripts`, `shaders`,
`sounds`, `spawns`, `textures`, `weapons`, and `weathers`. The script check parses emitted `.script` files with the
LuaJIT syntax dialect.

If `--ignore` is omitted, the command ignores common repository and unpacked-source entries: `.git`, `.idea`,
`particles_unpacked`, `textures_unpacked`, `.gitignore`, `.gitattributes`, `README.md`, and `LICENSE`.

## Checks and rules

A check is a group of related verification, selected with `--checks`. Inside a check, each individual violation is
attributed to a rule, and the rule identifier is what appears in findings and in the JSON report. Rule identifiers are
`<check>.<rule>`:

| Check             | Rules                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `animations`      | `animations.player-hud`, `animations.hud-item`, `animations.motion-collision`                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `levels`          | `levels.ai-guid`, `levels.ai-node-count`, `levels.ai-version`, `levels.cform-version`, `levels.details-pair`, `levels.file-empty`, `levels.file-truncated`, `levels.graph-duplicate`, `levels.graph-guid`, `levels.header-version`, `levels.level-guid`, `levels.ltx-read`, `levels.map-texture`, `levels.missing-bundle`, `levels.missing-file`, `levels.orphan-bundle`, `levels.roster-conflict`, `levels.shader-reference`, `levels.shaders-chunk`, `levels.texture-reference`, `levels.undeclared-map` |
| `ltx`             | `ltx.formatting`, `ltx.schema`, `ltx.verification`                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| `meshes`          | `meshes.path`, `meshes.read`, `meshes.validation`, `meshes.motion-read`, `meshes.motion-validation`, `meshes.shader-library`                                                                                                                                                                                                                                                                                                                                                                               |
| `particles`       | `particles.library`, `particles.texture`                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `particles-usage` | `particles-usage.reference`, `particles-usage.spawn`, `particles-usage.spawn-custom-data`                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `scripts`         | `scripts.path`, `scripts.read`, `scripts.syntax`                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `shaders`         | `shaders.renderer-root`, `shaders.lua-syntax`, `shaders.source-read`, `shaders.source-invalid`, `shaders.include-missing`, `shaders.include-cycle`, `shaders.include-syntax`                                                                                                                                                                                                                                                                                                                               |
| `sounds`          | `sounds.files`, `sounds.references`                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `spawns`          | `spawns.path`, `spawns.read`                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `textures`        | `textures.path`, `textures.read`, `textures.dds`, `textures.bump`                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `weapons`         | `weapons.validation`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `weathers`        | `weathers.definitions`, `weathers.files`, `weathers.validation`                                                                                                                                                                                                                                                                                                                                                                                                                                            |

`checks.execution` is reported when a check itself fails to run, rather than when content is invalid.

The animation rules validate player HUD and item motions. Missing item motions are allowed where the engine falls back
to `idle`; duplicate motion names across banks in one HUD namespace are reported because their resolution is ambiguous.

`textures.bump` resolves the bump each `.thm` declares the way `CTextureDescrMngr::LoadTHM` does, by the name in the
descriptor rather than by a `_bump` suffix convention. A name that resolves to nothing still takes the `_bump` shader
path, because `bump_exist()` only checks the name is non-empty: the loader substitutes `ed\ed_dummy_bump` and logs
`! Fallback to default bump map` on every load, so the surface is flat and the log is noisy. Importing a texture under a
new path is the usual way to produce one, since the copied descriptor keeps pointing into the source layout. Repoint it
with `thm patch-bump --to`, or `thm patch-bump --off` when the bump does not exist and is not going to.

## JSON report

Pass `--report` to write the result for CI or other tooling, or `--json` to put it on standard output for a pipe:

```powershell
xrf-cli gamedata verify ./target/gamedata --checks sounds,weathers --report ./verification-report.json
```

The file carries the [shared report envelope](cli.md#reporting); what this command found is under `result`:

```json
{
  "command": ["gamedata", "verify"],
  "duration": 311,
  "error": null,
  "exitCode": 0,
  "outcome": "success",
  "result": {
    "checks": [
      {
        "duration": 114,
        "findings": [],
        "status": "passed",
        "summary": "122/122 weather files valid",
        "verificationType": "weathers"
      }
    ],
    "duration": 311,
    "status": "passed"
  }
}
```

`status` is one of `passed`, `failed`, `error`, `incomplete`, or `skipped`. The top-level status is the most severe
individual status. `incomplete` means a check could cover only part of its expected input. Durations are whole
milliseconds, and a check's is `null` when it did not run.

Each entry of `findings` describes one violation:

| Field       | Meaning                                  |
| ----------- | ---------------------------------------- |
| `ruleId`    | Rule identifier from the table above.    |
| `assetPath` | Root-relative asset path. May be absent. |
| `message`   | Human-readable description.              |

Findings are ordered by asset path, rule, then message, so two reports over the same gamedata can be compared directly.

## Examples

```powershell
xrf-cli gamedata verify ./target/gamedata
xrf-cli gamedata verify ./target/gamedata --checks scripts,ltx
xrf-cli gamedata verify ./target/gamedata --checks weathers
xrf-cli gamedata verify ./target/gamedata --checks sounds --strict
xrf-cli gamedata verify ./target/gamedata --report ./verification-report.json
xrf-cli gamedata verify ./target/gamedata --ignore .git,textures_unpacked --strict
```

## Result

The command exits with a non-zero status unless the overall result is `passed`, including when verification is skipped
or incomplete. In normal logging mode it prints each failure message before exiting.

The command validates the files present in the assembled tree, including generated scripts and configs. It does not
validate source repositories or files that were not included in the build.

## Command reference

{{#include reference/gamedata.md:commands}}
