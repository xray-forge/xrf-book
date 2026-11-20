# THM CLI

A texture's `.thm` descriptor tells the engine and the SDK's converter what the texture is: its type, its format, its
detail and bump textures, and how its mip maps are built. `thm info` prints one. `thm write` creates one for a texture
that has none, and `thm set-field` changes fields on one descriptor or on every descriptor in a folder. `thm patch-bump`
changes or disables only the bump declaration; generate the bump texture files themselves with
[`dds make-bump`](dds.md#generate-a-bump-pair). `thm bump-alignment` checks whether each descriptor's bump belongs to
its diffuse, which a texture pack installed in part breaks without any file going missing.

The examples run from a gamedata root, the folder that holds `textures`, and use files of the unpacked game.

## Read a descriptor

```powershell
xrf-cli thm info ./textures/act/act_cat_bump.thm
```

Example output:

```text
Read thm file ./textures/act/act_cat_bump.thm
Kind: texture descriptor, version 0x0012
Type: Bump Map
Format: DXT3
Size: 512 x 512
Mip filter: Box
Flags: flDitherColor, flHasAlpha
Border colour: 00000000
Fade: colour 00000000, amount 0%, delay 0
Detail: '', scale 1
Material: Blin <-> Phong, weight 0
Bump: None '', virtual height 0.05
External normal map: ''
```

Exit code: `0`. Values are shown under the names the SDK's texture editor uses. The report carries the whole descriptor
under `result.file`, with `result.kind` telling a texture descriptor from an object, group or sound thumbnail, which it
reads too. A file that is not a thumbnail at all is exit `1`.

## Create a descriptor for a texture

A texture without a descriptor still loads, but the engine then binds no detail texture or bump to it, and the SDK has
no settings to rebuild it with. Give it one:

```powershell
xrf-cli thm write ./textures/mtl/mtl_fence_05.dds
```

This writes `mtl_fence_05.thm` beside the texture, starting from the values the SDK gives a new descriptor. Size and
format come from the DDS file: DXT1, DXT3 and DXT5 (BC1, BC2 and BC3) become `DXT1`, `DXT3` and `DXT5`, and an
uncompressed file the matching `32 bit`, `24 bit` or `16 bit` format. `flHasAlpha` is set when the format carries alpha.
A cube map's width is recorded as its six faces side by side, the strip the SDK builds a cube from, and, as the SDK
does, a cube map generates no mip maps.

A format no descriptor can name, such as BC7, needs `--format` to say which one to record. `--dest` writes elsewhere,
`--dry-run` reports the descriptor without writing it, and an existing descriptor is refused unless `--force` is given.
Any field `thm set-field` takes can be given here too. This one also declares the bump texture that sits beside it:

```powershell
xrf-cli thm write ./textures/mtl/mtl_walls7.dds --bump-mode use --bump-name "mtl\mtl_walls7_bump"
```

## Change fields

```powershell
xrf-cli thm set-field ./textures/wall --detail-scale 4 --dry-run
xrf-cli thm set-field ./textures/wall --detail-scale 4
```

Example output of the dry run:

```text
Dry run, nothing written; descriptors that would change: 94, already as named: 2, skipped: 0
```

Given a folder, `thm set-field` changes every texture descriptor below it and skips object, group and sound thumbnails.
Given one file, it changes that file, and refuses it if it is not a texture descriptor. Every other byte of each file
stays as it was: a descriptor that would not come back byte for byte is refused, and if any file in the run cannot be
read or kept that way, none is written. A descriptor that already holds the values asked for is not rewritten; the
report lists what `changed` and counts what was `unchanged` or `skipped`.

| Option                                           | Field                                                       |
| ------------------------------------------------ | ----------------------------------------------------------- |
| `--texture-type`                                 | `image`, `cube-map`, `bump-map`, `normal-map` or `terrain`  |
| `--format`, `--mip-filter`                       | What the SDK's converter builds the texture as              |
| `--detail-name`, `--detail-scale`                | The detail texture and how many times it repeats            |
| `--bump-mode`, `--bump-name`, `--virtual-height` | The bump declaration, as `thm patch-bump` sets it           |
| `--material`, `--material-weight`                | The shading pair and where between the two the texture sits |
| `--border-color`, `--fade-color`                 | Colours in hex, `AARRGGBB`                                  |
| `--fade-amount`, `--fade-delay`                  | How far each mip level fades, and from which level          |
| `--ext-normal-map`                               | The normal map the bump generator takes instead of its own  |
| `--set-flag`, `--clear-flag`                     | Flags by their SDK names, such as `flDiffuseDetail`         |

Each option sets exactly its field. Changing the type does not also change the flags, format and filter the SDK's editor
changes with it; name those too. A detail texture only shows when its flag is set as well:

```powershell
xrf-cli thm set-field ./textures/wall/wall_border_01.thm --detail-name "detail\detail_beton_det4" --set-flag flDiffuseDetail
```

## Repoint a bump declaration

Run from a gamedata root containing the descriptor and its replacement bump texture:

```powershell
xrf-cli thm patch-bump ./textures/wpn/wpn_pm/wpn_pm.thm --to "wpn\wpn_pm\wpn_pm_bump" --dry-run
xrf-cli thm patch-bump ./textures/wpn/wpn_pm/wpn_pm.thm --to "wpn\wpn_pm\wpn_pm_bump"
```

Example report result — preview a bump-name change:

```json
{
  "isDryRun": true,
  "originalSize": 138,
  "patchedSize": 154,
  "previousMode": 1,
  "previousName": ""
}
```

Exit code: `0`. This dry run changes no file. The report shows a 16-byte increase for the requested name.

The first command validates and reports the proposed change; the second rewrites the descriptor in place. The stored
name is relative to `textures`, uses backslashes, and omits the extension.

Use `--dest` to write a separate descriptor. `--to` changes the name while preserving the existing mode, including
`use_parallax`; it does not enable a descriptor whose mode is disabled.

## Disable a missing bump

If the surface should have no bump map, clear the declaration:

```powershell
xrf-cli thm patch-bump ./textures/tile/tile_walls_red_01.thm --off
```

`--off` sets the mode to `none` and clears the name, matching the form written by `STextureParams`. Give exactly one of
`--to` or `--off`: a command with neither or both is rejected as a usage error, exit 2.

## How the engine resolves it

`CTextureDescrMngr::LoadTHM` reads the descriptor beside the texture. An active bump declaration uses the stored name;
the engine does not discover a map merely because a neighboring file ends in `_bump.dds`.

If that name resolves to nothing, the renderer still selects the bump shader path and substitutes `ed\ed_dummy_bump`,
logging `! Fallback to default bump map`. The surface appears flat while retaining the bump rendering path. A copied
descriptor can cause this when it still points into the source project's texture layout. In `renderer_r4`, this lookup
happens through `CTexture::Preload`.

## Preservation and verification

The patcher rebuilds only the bump chunk and copies other chunks byte for byte. Before writing, it requires a rewrite of
the existing declaration to reproduce the source exactly. After writing, it reads the requested declaration back. A
failed read-back check triggers restoration of an in-place source or removal of a separate destination. This does not
make an interrupted or failed filesystem write transactional; use a separate destination when the original must remain
available.

From the directory containing the assembled `gamedata` tree, verify the resulting references:

```powershell
xrf-cli gamedata verify ./gamedata --checks textures --report ./texture-report.json
```

Resolve remaining bump findings before checking the surface in game. Descriptor validation establishes the reference; it
does not establish the visual quality of the map.

## Check that bumps belong to their diffuses

A bump drawn for one picture and laid under another still resolves, so `gamedata verify` passes it, but the relief no
longer lines up with what the surface shows: cracks and rivets light where the picture has none. This happens when a
texture pack replaces a diffuse and not its `_bump`, or the reverse. Compare every set an installation declares:

```powershell
xrf-cli thm bump-alignment ./stalker-anomaly --report ./alignment.json
```

Example output — stock Anomaly:

```text
Comparing the bumps texture descriptors declare with their diffuses
7132 descriptors, 2758 declaring a bump; 2661 sets compared, 2 missing a diffuse, 95 missing a bump, 0 unread
2241 aligned, 173 doubtful, 223 mismatched (98 with their own bump), 24 featureless; 1098 borrow another's bump
```

Exit code: `0`. It is a report rather than a check: the verdict is a measure's judgement, not a rule the engine
enforces.

Each set is compared by its edges: where the diffuse's colour changes against where the bump's relief changes, texel for
texel, at up to 512 texels a side. The same comparison with the bump shifted along the tile gives what any two textures
share by chance, and the verdict judges how far the set rises above it: `aligned`, `doubtful`, `mismatched`, or
`featureless` when either texture carries nothing to compare. A set whose bump is its diffuse's own `<texture>_bump` and
reads `mismatched` is the one to look at, and the console names each. A descriptor can also borrow another texture's
bump on purpose, a dirtier variant reusing the clean brick's relief; `isBorrowed` marks those in the report, and they
often follow their diffuse only loosely.

Layer a pack over the game it modifies to check the result as the game sees it, highest priority first:

```powershell
xrf-cli thm bump-alignment ./texture-pack ./stalker-anomaly --source auto --report ./alignment.json
```

`--prefix textures\wpn` limits the check to the descriptors of one folder; their bumps are still found wherever they
are.

## Command reference

{{#include reference/thm.md:commands}}
