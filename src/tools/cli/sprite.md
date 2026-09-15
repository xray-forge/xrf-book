# Sprite CLI

Sprite commands pack and unpack sheets containing multiple icons. Choose the workflow by the file that defines the
layout:

| Layout source                                           | Workflow                                    |
| ------------------------------------------------------- | ------------------------------------------- |
| `system.ltx` inventory sections and `inv_grid_*` fields | [Equipment sprite](#equipment-sprite)       |
| XML texture descriptions                                | [Description sprites](#description-sprites) |

Use [DDS commands](dds.md) to crop or convert an individual texture. The examples below run from an assembled gamedata
root containing `configs` and `textures`; packing writes the named output sheets.

## Equipment sprite

Unpack the equipment sheet into per-section images:

```powershell
xrf-cli sprite unpack-equipment --system-ltx ./configs/system.ltx `
  --source ./textures/ui/ui_icon_equipment.dds `
  --output ./textures_unpacked/ui/ui_icon_equipment
```

Only sections with `$inventory_icon = true` and all four fields—`inv_grid_x`, `inv_grid_y`, `inv_grid_width`, and
`inv_grid_height`—participate. Grid fields alone do not opt in a section, because abstract base sections commonly pass
them to descendants. `$inventory_icon = false` excludes a section.

Edit the extracted icons, check the grid, then pack the replacement sheet:

```powershell
xrf-cli sprite verify-equipment --system-ltx ./configs/system.ltx
xrf-cli sprite pack-equipment --system-ltx ./configs/system.ltx `
  --source ./textures_unpacked/ui/ui_icon_equipment `
  --output ./textures/ui/ui_icon_equipment.dds --strict
```

Packing prefers `<section>.png` over `<section>.dds`. Icons that differ from the target dimensions are fitted while
preserving aspect ratio and centered on a transparent canvas.

Without `--strict`, missing icon files are skipped with warnings. Strict packing reports all missing opted-in sections
and writes no sheet, allowing the full list to be corrected together. Use `--gamedata` when packing needs a separate
resource-lookup root.

Commands reading `system.ltx` accept `--dltx` for the [Monolith/Anomaly patch dialect](ltx.md#the-dltx-patch-dialect).
Select it when the layout depends on those patches.

### Checking the grid before moving an icon

Grid coordinates use 50 × 50 pixel cells. `verify-equipment` reports partial overlaps between sections, including the
shared cells and overlap count, and returns a non-zero exit code when it finds them.

Identical rectangles are allowed: variants such as `_nimble`, `_snag`, and `pri_a15_` quest copies often share their
base weapon's slot. Packing can warn when different art targets the same slot, but it does not replace the overlap
check.

A partial overlap is different: widening a `1 × 1` icon to `2 × 1` can cover a neighboring icon. Both may pack, with the
later write replacing shared pixels. Run verification before and after changing grid positions or dimensions, then
inspect the resulting sheet.

Example output — check the icon grid:

```text
Inventory icon grid is clean, no overlapping rects
```

Exit code: `0`.

## Description sprites

Unpack sheets named by an XML texture description:

```powershell
xrf-cli sprite unpack-description --description ./configs/ui/textures_descr/ui_actor.xml `
  --base ./textures --output ./textures_unpacked
```

Example output excerpt — unpack a sprite sheet:

```text
Unpacking for 1 files
Unpacked 1 files
```

Exit code: `0`.

After editing the extracted images, pack them back:

```powershell
xrf-cli sprite pack-description --description ./configs/ui/textures_descr/ui_actor.xml `
  --base ./textures_unpacked --output ./textures --strict
```

Each image must exactly match its declared rectangle; description packing does not rescale. Use
[DDS fitting](dds.md#fitting-into-different-bounds) when an imported icon needs different bounds.

The description and base path are required. Output defaults to the base path when omitted, so name a separate output
explicitly when keeping source and result apart. Both commands accept `--strict`; `-s` means `--silent`.

### Select sheets from a description

Both commands process every declared sheet by default. Repeat `--file` to narrow the selection:

```powershell
xrf-cli sprite pack-description `
  --description ./configs/ui/textures_descr/ui_actor_upgrades.xml --base ./textures_unpacked `
  --output ./textures --file ui_actor_weapons --strict
```

Use a declared path such as `ui\ui_actor_weapons`, with either separator, or an unambiguous bare name such as
`ui_actor_weapons`. Missing or ambiguous names are errors.

Unpacking distributes sheets across workers and accepts `--jobs`; packing is sequential. The engine repository wraps
common equipment and description workflows through `npm run cli -- sprites ...`.

## Command reference

{{#include reference/sprite.md:commands}}
