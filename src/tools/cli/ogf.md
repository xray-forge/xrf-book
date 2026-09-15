# OGF CLI

OGF commands inspect X-Ray models, check their dependencies, replace stored motion or texture references, and remove
recognized unread bytes. Use reference patching when relocating assets without re-exporting model geometry.

Examples run from a working directory containing the named `meshes` tree or model files. Patch commands rewrite the
source by default; `--dest` selects a separate output and `--dry-run` previews the change.

## `ogf info`

Inspect a model before changing its references:

```powershell
xrf-cli ogf info --path ./meshes/example.ogf
```

Example output excerpt — inspect model texture references:

```text
Bones: 1
[0] name: lod
[0] parent:
OGF children (1):
[0] texture name: wpn\wpn_pm
[0] shader name: models\model
```

Exit code: `0`.

The output includes available header and bounds data, textures and shaders, description metadata, bones and parents,
motion references, and progressive levels of detail. It also reports unparsed chunk ids and nested child visuals.

Progressive detail usually belongs to child visuals. Add `--verbose` to see each level's index-buffer offset, triangle
count, and vertex count.

Inspection is read-only. If parsing fails, confirm the file's source and game version, then compare the error with
neighboring models from the same archive.

## `ogf verify`

Check a model or directory before importing it:

```powershell
xrf-cli ogf verify --path ./meshes --root ./gamedata --report ./ogf-report.json
```

The command checks model data and resolves texture dependencies. Repeat `--root` for additional lookup roots, searched
after the visual's own tree. Missing dependencies can make verification incomplete; inspect the report before treating a
readable model as ready to use.

A passed check exits 0, invalid content exits 3, and error, incomplete, or skipped results exit 1. This is a data check;
inspect the model in the target renderer to verify its appearance.

## `ogf patch-motion-refs`

An animated model stores references to OMF animation banks. Replace the list when moving those banks:

```powershell
xrf-cli ogf patch-motion-refs --path ./meshes/wpn_ak74_hud.ogf `
  --refs "dynamics\weapons\wpn_ak74\wpn_ak74_hud_animation" --dry-run
xrf-cli ogf patch-motion-refs --path ./meshes/wpn_ak74_hud.ogf `
  --refs "dynamics\weapons\wpn_ak74\wpn_ak74_hud_animation"
```

Use backslashes and omit `.omf` for an individual bank. A reference ending in `\*.omf` loads every OMF in that
directory. Multiple values replace the list in the supplied order:

```powershell
xrf-cli ogf patch-motion-refs --path ./hands.ogf --dest ./hands.patched.ogf `
  --refs "first\animation" "second\animation"
xrf-cli ogf patch-motion-refs --path ./hands.ogf --dest ./hands.wildcard.ogf `
  --refs "dynamics\weapons\wpn_hand\hud_animation\*.omf"
```

Only the references chunk is rebuilt. The command preserves its existing form—an older comma-separated string or a newer
counted list—and copies geometry, bones, IK data, and other chunks byte for byte. A model without a references chunk is
refused.

Confirm the stored list with `ogf info`, then verify that the referenced banks exist and contain the required motions.

## `ogf patch-texture-refs`

Rename one exact texture reference, including occurrences in nested child visuals:

```powershell
xrf-cli ogf patch-texture-refs --path ./meshes/wpn_ak74u.ogf --from "wpn\wpn_aksu\wpn_aksu" `
  --to "wpn\wpn_ak74u\wpn_ak74u"
```

Names use backslashes and omit extensions. All matching texture chunks are rebuilt; paired shader names and unrelated
chunks are preserved. If `--from` matches nothing, the error lists the model's actual references.

Move the texture files, patch every model using the old name, then inspect the changed references with `ogf info` and
run [gamedata verification](gamedata.md). A model missed during the rename still points to the old path.

### Patch checks

Both reference patchers first apply the model's existing values and require byte-identical output. This guards against
losing chunks that cannot be reconstructed from parsed geometry.

After writing, the motion patcher requires the requested list to read back; the texture patcher requires the old name to
be absent and the new name present. A failed read-back check triggers restoration of an in-place source or removal of a
separate destination. Filesystem write interruptions are not covered by that check; use a separate destination when
retaining the original is required.

## `ogf fix`

Use `fix` for `meshes.chunk-residue` findings: some Anomaly and Call of Chernobyl models contain trailing bytes that the
engine does not consume, such as data beyond the counted motion-reference list.

Preview a directory sweep, then apply it or write a separate output for one model:

```powershell
xrf-cli ogf fix --path ./meshes --dry-run
xrf-cli ogf fix --path ./meshes/actors/stalker_zombied/stalker_zombied_bandit2a_face1.ogf
xrf-cli ogf fix --path ./meshes/wpn_m1891.ogf --dest ./fixed/wpn_m1891.ogf
```

Example output excerpt — remove unread model bytes:

```text
Fixing ogf visual ./gamedata/meshes/ogf/residue_split_motion_ref.ogf
Normalize ./gamedata/meshes/ogf/residue_split_motion_ref.ogf: 34 bytes the engine never reads
Ogf visual written into ./fixed.ogf
Normalized 1 of 1 visual(s), 34 bytes discarded, 0 unchanged, 0 failed
```

Stderr:

```text
Discarding uncounted motion reference 'actors\stalker_scenario_animation' from ./gamedata/meshes/ogf/residue_split_motion_ref.ogf
```

Exit code: `0`. This model contains 34 unread bytes. The source is retained and fixed.ogf receives the repaired model.

Directories are scanned recursively in path order. `--dest` is supported only for a single file; `--jobs` controls
directory-sweep parallelism.

### What changes

The fixer removes recognized unread motion-reference tails, adjusts the chunk's declared size, and removes accounted
trailing bytes after the last well-formed chunk. Other bytes are preserved. An already well-formed source is left
untouched; with `--dest`, a destination is still written.

Before writing, the normalized bytes must parse without residue and produce the same motion references. Unexplained
trailing data and chunks extending beyond the file are refused. The result is staged beside the destination and moved
into place.

A directory sweep continues after a refusal. Successfully fixed files remain changed; any refused files produce exit 1
and entries in `findings`. Check those entries, then use `ogf info` to confirm that repaired models no longer report
residue.

## Command reference

{{#include reference/ogf.md:commands}}
