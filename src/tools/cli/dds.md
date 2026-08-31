# DDS CLI

DDS commands inspect and cut single texture files. Sheet-level work belongs to the [sprite commands](sprite.md).

## Commands

| Command          | Purpose                                                            |
| ---------------- | ------------------------------------------------------------------ |
| `dds info`       | Print DDS size, metadata, mipmap, format, and compression details. |
| `dds crop`       | Crop a pixel region out of a DDS file into a new DDS or PNG file.  |
| `thm patch-bump` | Repoint the bump texture a `.thm` descriptor declares.             |

## DDS inspection

```powershell
xrf-cli dds info --path ./textures/ui/ui_icon_equipment.dds
```

The command prints file size, metadata size, pixel data size, dimensions, mipmap information, pitch or linear size when
present, block size, bits per pixel, FourCC, and D3D/DXGI format when known.

## Region cropping

```powershell
xrf-cli dds crop --source ./textures/ui/ui_icon_equipment.dds --output ./wpn_ak74.dds --x 1000 --y 0 --width 250 --height 100
```

`--source`, `--output`, `--x`, `--y`, `--width`, and `--height` are required. Coordinates start at the top left. An
out-of-bounds region is rejected rather than truncated. The command also accepts `-s, --silent` and `-v, --verbose`.

The `.png` extension selects lossless PNG; any other name produces a BC3 DDS. Prefer PNG as a later packing source to
avoid another lossy decode and encode. `sprite pack-equipment` prefers `<section>.png` over `<section>.dds`.

Use this to lift a single icon out of a sprite sheet when the sheet belongs to another project, where
`sprite unpack-equipment` cannot help because that project's configs do not declare `$inventory_icon`.

### Fitting into different bounds

`--fit-width` and `--fit-height` scale the cropped region after cropping. They must be given together.

```powershell
xrf-cli dds crop --source ./ui_actor_weapons.dds --output ./upgrade_ak74.dds --x 0 --y 400 --width 300 --height 100 --fit-width 295 --fit-height 110
```

Scaling preserves aspect ratio and centers the result on a transparent canvas instead of stretching it. A matching
region is unchanged. `sprite pack-equipment` uses the same fitting for mismatched icons.

Fitting matters where a target rectangle is fixed by an XML texture description, since `sprite pack-description`
requires the file to match the declared size exactly and does not rescale.

## Bump declarations

A texture's bump map is not found by naming convention. `CTextureDescrMngr::LoadTHM` reads the `.thm` beside the texture
and takes its bump name verbatim.

A name that resolves to nothing does not turn bump mapping off. `bump_exist()` only tests that the name is non-empty, so
the renderer still selects the `_bump` shader variant, and the loader substitutes `ed\ed_dummy_bump` while logging
`! Fallback to default bump map: <name>` on every load. The surface renders flat while still paying for the bump path,
and the log fills with fallbacks.

This bites whenever a texture is imported under a different path than it had at its source, because the copied
descriptor keeps pointing into the source layout. On `renderer_r4` the bump is consumed through `CTexture::Preload`, so
it is not a setting the player can opt out of.

```powershell
xrf-cli thm patch-bump --path ./textures/wpn/wpn_pm/wpn_pm.thm --to "wpn\wpn_pm\wpn_pm_bump"
```

`--path` is required, plus either `--to` or `--off`. `--dest` writes elsewhere instead of rewriting in place,
`--dry-run` reports what would change without writing, and `-s, --silent` and `-v, --verbose` control logging. The bump
name is engine style, backslash separated and without an extension.

For a bump that does not exist and is not going to, declare none rather than leaving a dangling name:

```powershell
xrf-cli thm patch-bump --path ./textures/tile/tile_walls_red_01.thm --off
```

`--off` sets mode to `none` and clears the name, which is the form `STextureParams` itself writes, so the result stays
diffable against vanilla. Prefer it over leaving the name in place: a dangling name still selects the bump shader path
and still logs a fallback on every load.

Omitted fields keep their existing value, so `--to` alone preserves a `use_parallax` mode across a rename.

The patch rewrites only the bump chunk and copies every other chunk byte for byte. It first proves that rewriting the
declaration the file already has reproduces the source exactly, then confirms the written file reads the requested name
back, and reverts rather than leaving a half-written descriptor behind.

Run `gamedata verify --checks textures` afterwards: it resolves every declared bump the same way the engine does and
reports the ones that point at nothing.

## Command reference

{{#include reference/dds.md:commands}}
