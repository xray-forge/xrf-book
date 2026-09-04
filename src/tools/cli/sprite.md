# Sprite CLI

Sprite commands pack and unpack the sheets the engine reads as one texture. A sprite is the whole sheet; an icon is one
image or rectangle inside it. Single-file DDS work belongs to the [dds commands](dds.md).

Two families of sheet exist, told apart by what declares the layout. The **equipment sprite** is laid out by the
`inv_grid_*` fields of `system.ltx`. **Description sprites** are laid out by an XML texture description.

## Commands

| Command                     | Purpose                                                                      |
| --------------------------- | ---------------------------------------------------------------------------- |
| `sprite unpack-equipment`   | Slice an equipment sprite into per-section icon files using `system.ltx`.    |
| `sprite pack-equipment`     | Pack per-section icon files into an equipment DDS sprite using `system.ltx`. |
| `sprite verify-equipment`   | Report inventory icon grid rects that overlap each other.                    |
| `sprite unpack-description` | Slice the sprites an XML texture description declares.                       |
| `sprite pack-description`   | Pack the sprites an XML texture description declares.                        |

## Equipment sprite

```powershell
xrf-cli sprite unpack-equipment --system-ltx ./configs/system.ltx --source ./textures/ui/ui_icon_equipment.dds --output ./textures_unpacked/ui/ui_icon_equipment
xrf-cli sprite pack-equipment --system-ltx ./configs/system.ltx --source ./textures_unpacked/ui/ui_icon_equipment --output ./textures/ui/ui_icon_equipment.dds --strict
```

Every command that reads `system.ltx` accepts `--dltx`, which resolves it with the Monolith/Anomaly patch dialect so the
icon grid matches what a patched install actually declares. See [LTX CLI](ltx.md#the-dltx-patch-dialect).

`sprite pack-equipment` also accepts `--gamedata <path>` for resource lookup, plus `-v, --verbose` and `-s, --strict`.
`sprite unpack-equipment` supports `-v, --verbose`.

Both commands act only on sections with `$inventory_icon = true` and all four grid fields: `inv_grid_x`, `inv_grid_y`,
`inv_grid_width`, and `inv_grid_height`. Grid fields alone do not opt a section in; `$inventory_icon = false` excludes
it.

The explicit flag matters because inherited grid fields are common on abstract base sections that have no icon of their
own.

Without `--strict`, an opted-in section whose icon file is missing is skipped with a warning. With `--strict` the
command packs nothing and reports every such section in a single error, so the whole list can be fixed in one pass.

### Checking the grid before moving an icon

```powershell
xrf-cli sprite verify-equipment --system-ltx ./configs/system.ltx
```

Reports every pair of sections whose grid rects cover a shared cell, with the cell and how many cells overlap, and exits
non-zero when any are found.

This covers a case packing cannot. `sprite pack-equipment` warns when two sections write different art to the **same**
slot, which is the common and usually harmless case, because variants such as `_nimble`, `_snag` and the `pri_a15_`
quest copies inherit their base weapon's position by design. Identical rects are therefore not reported here either.

What it catches instead is a rect reaching **into** a neighbour: widen a `1x1` icon to `2x1` and it may quietly take a
cell another weapon already occupies. Both pack without complaint, and whichever writes last wins those pixels, so the
loser silently shows the wrong art. Run this before widening or relocating any icon, and again afterwards.

Grid coordinates are cells, not pixels; a cell is 50x50, hardcoded in the engine.

## Description sprites

```powershell
xrf-cli sprite unpack-description --description ./configs/ui/textures_descr/ui_actor.xml --base ./textures --output ./textures_unpacked
xrf-cli sprite pack-description --description ./configs/ui/textures_descr/ui_actor.xml --base ./textures_unpacked --output ./textures --strict
```

Description commands require `--description` and `--base`. If `--output` is omitted, output defaults to the base path.
Both support `-v, --verbose` and `-s, --strict`. Unpacking spreads its sheets across workers and takes `-j, --jobs`;
packing is sequential and does not.

A description can name several sheets; both commands rewrite all of them by default. Repeat `--file <name>` to select
one or more. Use the declared path (`ui\ui_actor_weapons`) with either separator, or an unambiguous bare name
(`ui_actor_weapons`). Missing and ambiguous names are errors.

```powershell
xrf-cli sprite pack-description --description ./configs/ui/textures_descr/ui_actor_upgrades.xml --base ./textures_unpacked --output ./textures --file ui_actor_weapons --strict
```

The engine repository wraps the common equipment and description workflows through `npm run cli -- sprites ...`.

## Command reference

{{#include reference/sprite.md:commands}}
