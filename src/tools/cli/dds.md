# DDS CLI

DDS commands inspect textures, crop regions, convert image formats, and generate X-Ray bump maps. Use
[sprite commands](sprite.md) when a config or XML description defines a whole sheet, and [THM commands](thm.md) to
change a texture's bump declaration.

Examples use paths relative to a gamedata or texture-working directory. Output commands write the named files; choose
separate output paths when keeping the originals.

## DDS inspection

```powershell
xrf-cli dds info --path ./textures/ui/ui_icon_equipment.dds
```

The report includes dimensions, mipmaps, file and pixel-data sizes, compression, block size, bits per pixel, and known
FourCC or D3D/DXGI formats. Pitch or linear size is included when present. Inspect these fields before selecting an
output format or diagnosing a texture that the renderer cannot load.

## Region cropping

Crop a single icon when its source sheet has no compatible inventory config:

```powershell
xrf-cli dds crop --source ./textures/ui/ui_icon_equipment.dds --output ./wpn_ak74.png `
  --x 1000 --y 0 --width 250 --height 100
```

Coordinates and dimensions are pixels, measured from the top left. The source must contain the entire rectangle;
out-of-bounds regions are rejected.

A `.png` output preserves the decoded pixels without another lossy encode. Any other output extension selects BC3 DDS.
Prefer PNG for subsequent packing; `sprite pack-equipment` selects `<section>.png` before `<section>.dds`.

### Fitting into different bounds

Supply both fit dimensions to resize the crop into a fixed output rectangle:

```powershell
xrf-cli dds crop --source ./ui_actor_weapons.dds --output ./upgrade_ak74.png --x 0 --y 400 `
  --width 300 --height 100 --fit-width 295 --fit-height 110
```

Fitting preserves aspect ratio and centers the image on a transparent canvas. A crop already matching the requested
bounds is unchanged. Equipment packing uses the same fitting behavior; description packing instead requires exact
dimensions, so fit those icons before packing.

## Convert a texture

Re-encode an existing DDS texture in an explicit format:

```powershell
xrf-cli dds convert ./source.dds ./texture.dds --format bc3
xrf-cli dds info --path ./texture.dds
```

Accepted formats are `bc1`, `bc2`, `bc3`, `bc7`, and `rgba8`. Choose a format supported by the target renderer and
appropriate for the texture's alpha and quality requirements.

Conversion decodes the base image and rebuilds its mip chain. Use `--no-mipmaps` to write only the base level.
`--mip-filter` selects the reduction filter, defaulting to `kaiser`; `--quality` trades encoding time for fidelity,
defaulting to `slow`.

Add `--compare` to encode the other formats in memory and report their size and distortion alongside the selected
format. Only the selected format is written. Review the resulting texture visually as well as inspecting its metadata;
numerical error alone does not establish acceptable appearance.

## Generate a bump pair

From a working directory containing a height image, generate the two DDS files used by an X-Ray bumped surface:

```powershell
xrf-cli dds make-bump ./height.png ./textures/tile/wall --gloss-constant 0.5
```

The destination is a base path without an extension or `_bump` suffix. This example writes `textures/tile/wall_bump.dds`
and `textures/tile/wall_bump#.dds`.

Height is averaged across the input's color channels. Supply `--gloss` for a gloss-mask image instead of a constant
between 0 and 1. An optional `--normal-map` supplies normals instead of deriving them from height; its dimensions must
match the height image. `--virtual-height` controls relief depth, defaulting to `0.05`.

Bump generation uses the `box` mip filter by default. A warning about very dark gloss indicates little specular
response; the files are still written because a matte surface may be intentional.

## Bump declarations

Generating or moving a bump texture does not update its descriptor. Follow
[the THM bump-declaration workflow](thm.md#repoint-a-bump-declaration) to connect an existing descriptor to the new
path, then verify the assembled texture set.

## Command reference

{{#include reference/dds.md:commands}}
