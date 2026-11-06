# Object CLI

`object decompile` turns a compiled OGF model back into the SDK `.object` it was exported from, so the Actor Editor can
open it again. Use it when a model ships without its source and needs editing. The `.object` gets a provenance sidecar
beside it, so a decompiled file is never taken for an authored one.

`object compile` goes the other way, writing the OGF the Actor Editor's **Export OGF** would write, without the editor.

## Compile a model

```powershell
xrf-cli object compile --path ./rawdata/objects/dynamics/workshop_room/hammer.object --dest ./meshes/dynamics/workshop_room/hammer.ogf
```

Example output — the vanilla hammer:

```text
Compiled ./rawdata/objects/dynamics/workshop_room/hammer.object into ./meshes/dynamics/workshop_room/hammer.ogf: 1 children, 56 vertices, 32 faces (0 dropped), 2 bones
```

Exit code: `0`. The command refuses to replace an existing OGF; pass `--force` to overwrite it.

Skeleton objects compile: props and physics objects, weapon hands and characters, with the motions they keep themselves
(a fan's spin, a door opening) or the motion files they reference. A static object is refused with exit `1`, and so is a
motion that moves the whole object through its root bone. So is a face spanning two breakable parts, which the editor's
export cannot split either.

The OGF draws as the editor's would. Against 134 vanilla models compiled from their SDK sources, every face, nearly
every vertex, normal and weight, and over 99% of tangents matched the editor's. Bones, physics, class script, LODs and
motion references matched exactly, and compiled props load, draw and collide in game. It is not the same bytes. The
children can come in another order, the triangles are in the build's own order, and each bone's box is fitted along the
bone's axes, never smaller than the editor's.

## Decompile a model

```powershell
xrf-cli object decompile --path ./meshes/ventilator_01.ogf --dest ./rawdata/ventilator_01.object
```

Example output — a vanilla fan:

```text
Decompiled ./meshes/ventilator_01.ogf into ./rawdata/ventilator_01.object: 1 surfaces, 414 points, 726 faces (726 two-sided), 4 bones; provenance in ./rawdata/ventilator_01.object.xrf.json
```

Exit code: `0`. The command refuses to replace an existing `.object` or sidecar; pass `--force` to overwrite both.

Skeleton models decompile: props and physics objects, weapon hands and characters, whose vertices follow one bone or
blend across up to four. A static level-style model is refused with exit `1`.

## What comes back

The OGF keeps most of what the editor exported, and the decompiler restores it:

- **Points and faces.** The export split a point wherever two faces met at a UV seam or a hard edge; the decompiler
  welds corners with the same position and bones back into one point, so the mesh edits as one piece.
- **Bone weights.** Each point gets an entry in the weight map of every bone it follows. The weights come back as the
  export stored them: the four strongest, scaled to sum to one, which is what the editor exports from in any case.
- **Texture coordinates**, with a seam's second UV stored per face as the editor keeps it.
- **Two-sided surfaces.** The export wrote each face of a two-sided surface twice, the copy facing the other way; each
  pair becomes one face on a surface with the two-sided flag.
- **Smoothing.** Normals are recomputed in the editor from per-edge smoothing flags. An edge between two faces is soft
  when their stored normals agree at both its ends, and hard otherwise, so the editor shades the model as the OGF did.
- **Bones**, with their bind pose, shapes, joint types and limits, mass and friction, and the motion references, class
  script, LODs and description the OGF carries.

## What is inferred

Some of a source file never reaches the OGF, and the decompiler writes the editor's defaults instead:

- surface names, taken from the texture's file name;
- each surface's compiler shader and game material, `default`;
- the mesh name, `skin`;
- bone lengths, `0.5`.

The sidecar lists each of these under `inferred`. Check the game materials in particular before rebuilding a model whose
physics sound or particles matter.

The export also merged points closer than a millimetre, so a decompiled model can hold a few fewer points than its
source did where two parts nearly touched.

## The provenance sidecar

`<name>.object.xrf.json` records where the file came from:

```json
{
  "source": "./meshes/ventilator_01.ogf",
  "sourceSha256": "9e903f79d00ec2bbdb8070d37f487d852a71124d636b013fde0d8e60c05e200d",
  "tool": "xrf-cli object decompile",
  "toolVersion": "0.1.0 (d8e8327)",
  "decompiledAt": 1793940803,
  "inferred": [
    "surfaceNames",
    "surfaceCompilerShaders",
    "surfaceGameMaterials",
    "meshName",
    "boneLengths",
    "smoothing",
    "twoSidedSurfaces"
  ]
}
```

`sourceSha256` identifies the exact OGF, and `decompiledAt` is in Unix seconds. The SDK ignores the sidecar; keep it
with the `.object` while the file stays a decompile.

## Command reference

{{#include reference/object.md:commands}}
