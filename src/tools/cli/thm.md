# THM CLI

`thm patch-bump` changes or disables the bump declaration in an existing texture descriptor. Use it when moving an
imported texture or correcting a missing bump reference. Generate the texture files separately with
[`dds make-bump`](dds.md#generate-a-bump-pair). `thm bump-alignment` checks whether each descriptor's bump belongs to
its diffuse, which a texture pack installed in part breaks without any file going missing.

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
xrf-cli gamedata verify --path ./gamedata --checks textures --report ./texture-report.json
```

Resolve remaining bump findings before checking the surface in game. Descriptor validation establishes the reference; it
does not establish the visual quality of the map.

## Check that bumps belong to their diffuses

A bump drawn for one picture and laid under another still resolves, so `gamedata verify` passes it, but the relief no
longer lines up with what the surface shows: cracks and rivets light where the picture has none. This happens when a
texture pack replaces a diffuse and not its `_bump`, or the reverse. Compare every set an installation declares:

```powershell
xrf-cli thm bump-alignment --path ./stalker-anomaly --report ./alignment.json
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
xrf-cli thm bump-alignment --path ./texture-pack ./stalker-anomaly --source auto --report ./alignment.json
```

`--prefix textures\wpn` limits the check to the descriptors of one folder; their bumps are still found wherever they
are.

## Command reference

{{#include reference/thm.md:commands}}
