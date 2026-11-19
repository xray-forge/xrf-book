# Object CLI

`object decompile` turns a compiled OGF model back into the SDK `.object` it was exported from, so the Actor Editor can
open it again. Use it when a model ships without its source and needs editing. The `.object` gets a provenance sidecar
beside it, so a decompiled file is never taken for an authored one.

`object compile` goes the other way, writing the OGF the Actor Editor's **Export OGF** would write, without the editor.
`object verify` compiles an object in memory and compares the result with an OGF it should stand in for, such as the one
shipped in `gamedata`, before the OGF is replaced.

`object decompile` and `object compile` also handle `.dm` detail models, the small meshes such as lightning bolts the
game draws as details: see [Detail models](#detail-models). `object verify` compares OGFs only.

## Compile a model

```powershell
xrf-cli object compile ./rawdata/objects/dynamics/workshop_room/hammer.object --dest ./meshes/dynamics/workshop_room/hammer.ogf
```

Example output — the vanilla hammer:

```text
Compiled ./rawdata/objects/dynamics/workshop_room/hammer.object into ./meshes/dynamics/workshop_room/hammer.ogf: 1 children, 56 vertices, 32 faces (0 dropped), 2 bones, Actor Editor tangents
```

Exit code: `0`. The command refuses to replace an existing OGF; pass `--force` to overwrite it.

`--tangents` picks how vertices get the tangents normal maps are read against:

- **`auto`** (the default): a decompiled object compiles with the method its source model was made with, which the
  decompiler records in the sidecar beside it: MikkTSpace for a model exported from Blender, the editor's otherwise. An
  object with no sidecar gets the editor's.
- **`actor-editor`**: the editor's own tangents, which vanilla and SDK-made models carry.
- **`mikktspace`**: MikkTSpace, as Blender, Substance and similar tools bake normal maps against. Use it for a model
  made in Blender that has no sidecar.

Each gives back the tangents of the models made its way on most vertices (MikkTSpace over 95% on blender-xray models,
where the editor's matches under half) and does worse on the others, so neither replaces the other.

Skeleton objects compile: props and physics objects, weapon hands and characters, with the motions they keep themselves
(a fan's spin, a door opening) or the motion files they reference. A motion that moves the whole object through its root
bone is refused with exit `1`, and so is a face spanning two breakable parts, which the editor's export cannot split
either.

Static objects compile too. As in the editor, any object without bones exports static, even one flagged dynamic: one
surface becomes the model itself, several become a model with a part per surface. Static models store no tangents, so
`--tangents` does not apply, and the report's `tangents` is `null`. A progressive object compiles at full detail,
without the levels of detail the editor builds for it.

The OGF draws as the editor's would. Against 134 vanilla models compiled from their SDK sources, every face, nearly
every vertex, normal and weight, and over 99% of tangents matched the editor's. Bones, physics, class script, LODs and
motion references matched exactly, and compiled props load, draw and collide in game. It is not the same bytes. The
children can come in another order, the triangles are ordered for the GPU's vertex cache by a modern optimiser rather
than the editor's (as few cache misses, often fewer), and each bone's box is fitted tightly around every vertex the bone
moves, smaller than the editor's in sum and holding the model better as it animates. A two-sided surface of a static
object is written once, where the editor writes all of its faces twice over by mistake; the model draws the same.

## Verify a model

```powershell
xrf-cli object verify ./rawdata/objects/dynamics/balon/bidon.object --ogf ./meshes/dynamics/balon/bidon.ogf
```

Example output — the vanilla canister against its shipped OGF:

```text
Verifying ./rawdata/objects/dynamics/balon/bidon.object against ./meshes/dynamics/balon/bidon.ogf
Children 1 against 1; vertices matched 1922 of 1922 (0 more but for their normal); reference faces drawn 1032 of 1032 (0 with two corners at one position)
Verification of the object: passed
```

Exit code: `0` when nothing differs, `3` when something does. Nothing is written. The comparison covers what the engine
reads, in four checks:

- **`object.model`**: the model type, the user data (the class script), the LODs and the motion references.
- **`object.skeleton`**: every bone's name, parent and order, and its physics: game material, shape, joint, mass, centre
  of mass and bind pose.
- **`object.motions`**: the model's own motions by name and length, and the parameters and bone parts they play by.
- **`object.geometry`**: the bounds, and what each texture and shader draws, over every child drawing with them, against
  the OGF's: its vertices (position, normal, texture coordinate and weights) and triangles.

The order of children, vertices and triangles, how a surface's faces fall among children (the SDK splits one by
breakable part where older exporters kept it whole), the tangents, the bones' boxes and the description's authoring
stamps are left out, as the compile builds them its own way. A face with two corners at one position draws nothing, so
the OGF's faces of that kind are counted apart and never missed.

Pass `--verbose` to list each difference. A vertex that differs in its normal alone is counted as such: the model draws
and moves alike but shades differently. Expect it for an SDK source exported by an older editor that read smoothing
another way; a decompiled object keeps the normals it was decompiled from:

```text
Children 1 against 1; vertices matched 246 of 1083 (837 more but for their normal); reference faces drawn 594 of 594 (0 with two corners at one position)
Check 'object.geometry' is failed with 1 finding(s)
  [object.geometry] -: prop\prop_ballon1 / models\model: 837 of 1083 vertices have no reference vertex like them, 837 of them differing in their normal alone
Check failed: 1 finding(s)
```

An object the compile refuses is refused here too, with exit `1`.

## Decompile a model

```powershell
xrf-cli object decompile ./meshes/ventilator_01.ogf --dest ./rawdata/ventilator_01.object
```

Example output — a vanilla fan:

```text
Decompiled ./meshes/ventilator_01.ogf into ./rawdata/ventilator_01.object: 1 surfaces, 414 points, 726 faces (726 two-sided), 4 bones; provenance in ./rawdata/ventilator_01.object.xrf.json
```

Exit code: `0`. The command refuses to replace an existing `.object` or sidecar; pass `--force` to overwrite both.

Skeleton models decompile: props and physics objects, weapon hands and characters, whose vertices follow one bone or
blend across up to four. So do static models, a single one or one made of parts, into a static object without bones; a
progressive one comes back at its full detail, marked progressive. Other model types, such as trees, particles and level
geometry, are refused with exit `1`.

## What comes back

The OGF keeps most of what the editor exported, and the decompiler restores it:

- **Points and faces.** The export split a point wherever two faces met at a UV seam or a hard edge; the decompiler
  welds corners with the same position, bones and weights back into one point, so the mesh edits as one piece. Two
  points the export moved onto one position stay two, each with its own weights.
- **Bone weights.** Each point gets an entry in the weight map of every bone it follows. The weights come back as the
  export stored them: the four strongest, scaled to sum to one, which is what the editor exports from in any case.
- **Texture coordinates**, with a seam's second UV stored per face as the editor keeps it.
- **Two-sided surfaces.** The export wrote each face of a two-sided surface twice, the copy facing the other way; each
  pair becomes one face on a surface with the two-sided flag. The editor's static export writes both again by mistake;
  those repeats are dropped.
- **Normals and smoothing.** Each face corner keeps the normal the OGF stored, in the mesh chunk blender-xray also
  writes and reads, so a compile with xrf-cli and an import into Blender shade the model exactly as before. The SDK's
  editors ignore that chunk and recompute normals from per-edge smoothing flags, which the decompiler also writes: an
  edge between two faces is soft when their stored normals agree at both its ends, and hard otherwise. Models made in
  Blender weight their normals differently, so the SDK shades them slightly otherwise than the OGF did.
- **Bones**, with their bind pose, shapes, joint types and limits, mass and friction, and the motion references, class
  script, LODs and description the OGF carries.
- **Motions** a model keeps itself, a fan's spin or a door opening, with their speed, blending, marks and bone parts.
  Each bone is keyed at every frame, so the motion plays as it did; compiled again, every key lands within a
  quantisation step of the original.

## What is inferred

Some of a source file never reaches the OGF, and the decompiler writes the editor's defaults instead:

- surface names, taken from the texture's file name;
- each surface's compiler shader and game material, `default`;
- the mesh name, `skin` for a skeleton, `mesh` for a static object, which the editor names after the object;
- a skeleton's bone lengths, `0.5`;
- a motion's curves between its frames, which come back as straight lines from frame to frame.

The sidecar lists each of these under `inferred`. Check the game materials in particular before rebuilding a model whose
physics sound or particles matter.

The export also merged points closer than a millimetre, so a decompiled model can hold a few fewer points than its
source did where two parts nearly touched.

## Detail models

A `.dm` path decompiles a detail model into an object of one mesh and one surface, and a `.dm` destination compiles an
object into one, as the Level Editor's detail export writes it:

```powershell
xrf-cli object decompile ./meshes/dm/dm_lightning-01.dm --dest ./rawdata/dm_lightning-01.object
xrf-cli object compile ./rawdata/dm_lightning-01.object --dest ./meshes/dm/dm_lightning-01.dm
```

The vanilla lightning models come back exactly: decompiled and compiled again, each is the same file, byte for byte. A
detail model stores no normals, so the object's edges all come back smooth and no surface two-sided; the compile needs
neither. A model's flags (whether the wind sways it) and the range its planted copies are scaled in are not part of an
object: the decompile records them in the sidecar, and the compile writes them back from there, or uses the editor's
defaults (no flags, 0.5 to 2) for an object without one.

To compile, the object needs exactly one surface; the compile uses its first mesh, as the editor does. Faces that repeat
another, a back face among them, are dropped, and corners within a millimetre of each other become one vertex.

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
    "twoSidedSurfaces",
    "motionCurves"
  ]
}
```

`sourceSha256` identifies the exact OGF, and `decompiledAt` is in Unix seconds. The SDK ignores the sidecar; keep it
with the `.object` while the file stays a decompile.

## Command reference

{{#include reference/object.md:commands}}
