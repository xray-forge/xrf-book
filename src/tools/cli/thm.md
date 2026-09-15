# THM CLI

`thm patch-bump` changes or disables the bump declaration in an existing texture descriptor. Use it when moving an
imported texture or correcting a missing bump reference. Generate the texture files separately with
[`dds make-bump`](dds.md#generate-a-bump-pair).

## Repoint a bump declaration

Run from a gamedata root containing the descriptor and its replacement bump texture:

```powershell
xrf-cli thm patch-bump --path ./textures/wpn/wpn_pm/wpn_pm.thm --to "wpn\wpn_pm\wpn_pm_bump" `
  --dry-run
xrf-cli thm patch-bump --path ./textures/wpn/wpn_pm/wpn_pm.thm --to "wpn\wpn_pm\wpn_pm_bump"
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
xrf-cli thm patch-bump --path ./textures/tile/tile_walls_red_01.thm --off
```

`--off` sets the mode to `none` and clears the name, matching the form written by `STextureParams`. Choose either `--to`
or `--off`; they cannot be combined.

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

## Command reference

{{#include reference/thm.md:commands}}
