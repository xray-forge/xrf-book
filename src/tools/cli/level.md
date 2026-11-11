# Level CLI

Level commands read one compiled level: the `level` bundle with its shader table, sectors, portals, lights and visuals,
and the `level.geom` and `level.geomx` buffers the visuals draw from. Every command but `level decompile` names the
level the same two ways: `--path` is a level directory on disk, and `--root` with `--level` is one of an installation's
levels, read through the game's file system by its engine name `levels\<name>`. A stock Anomaly installation ships its
levels inside `.db` volumes, with no directory to point at, so the second way is the one that reads a real game.
`--root` without `--level` fails and names the levels the roots hold.

## Examples

Print what a level holds:

```powershell
xrf-cli level info --root ./stalker-anomaly --level zaton
```

```text
Read compiled level levels\zaton
Built by xrLC version 14, quality 1
97 sectors, 420 portals, 55 static lights, one of them the sun
Shader table entries: 922
Visuals: 25935, drawable: 17854, with fast path: 7257
level.geom: 41 vertex buffers, 16 index buffers, 131 slide windows
level.geomx: 19 vertex buffers, 11 index buffers, 0 slide windows
```

Verify that every drawable visual can be drawn the way the renderer addresses it:

```powershell
xrf-cli level verify --root ./stalker-anomaly --level zaton
```

`level verify` fails with exit code `3` on a range past a buffer, an index past the visual that draws it, a vertex
declaration the level compiler does not write, a sector or portal naming nothing, or a portal that is no polygon.

Measure the level's textures as albedo, the colour a surface reflects:

```powershell
xrf-cli level albedo --root ./stalker-anomaly --level zaton --report zaton-albedo.json
```

```text
Measuring the albedo of compiled level levels\zaton
921 surfaces, 42 writing no albedo; 330 base textures, 325 measured
Against linear 0.03 to 0.9: 95 dark, 230 within, 0 bright; median albedo 0.057, median coarse spread 1.20 stops
```

Exit code: `0`.

Decompile a level directory into a Level Editor scene:

```powershell
xrf-cli level decompile --path ./gamedata/levels/zaton --dest ./rawdata/levels/zaton.level
```

```text
Static geometry: 118 objects under ./rawdata/objects, 1648157 faces from 1673272 triangles (25115 two-sided folds, 0 repeats dropped); 1567425 tree triangles left for their own objects
Entities from ./gamedata/spawns/all.spawn (1866 objects): 0 disagreements with level.spawn and 262 update packets recorded in the sidecar
Decompiled ./gamedata/levels/zaton into ./rawdata/levels/zaton.level from 10 files: fog_volume 0, glow 36, light 571, portal 495, ps 129, scene_object 118, sector 118, sound_src 12, spawn 2183, wallmark, way 1506; provenance in ./rawdata/levels/zaton.level.xrf.json
```

## Decompile

`level decompile` turns the files the SDK's Level Editor writes straight from a scene, with no compiler in between, back
into that scene's parts. That covers static particles, sounds and sound environments, fog volumes, wall marks, patrol
paths, respawn points, environment modifiers and entity spawns, and also the glows and lights the `level` file and
`build.lights` keep. It reads a level directory on disk only, not a level inside `.db` volumes.

Where the directory holds `level.geom`, the static geometry comes back as one `.object` per compiled sector,
`<name>_sector_<N>`, with the sectors and portals that cull it:

- The compiled level merged every authored object of a sector into one mesh, so each sector's static triangles come back
  as one object, welded, with mirrored triangles folded into two-sided faces. Trees are not written yet.
- Each object keeps the level's own coordinates and stands at the origin, so every point lands exactly where the level
  had it and neighbouring sectors still meet.
- Collision faces in `level.cform` that no render triangle draws, such as invisible walls, come back as one more object,
  `<name>_collision`, with one mesh per sector. It is dressed as GSC dressed such props: the `prop\prop_fake_kollision`
  texture and the `def_shaders\def_kolizion_vertex` compiler shader, which collides and never renders. Each face keeps
  its game material. Collision under trees is left for the trees.
- The Build also bakes every wall mark into the static geometry. Those copies are left out of the shells, since
  `wallmark.part` holds the wall marks and a rebuild would otherwise draw them twice.
- `level.hom`, the occlusion mesh, comes back as `<name>_hom`, as GSC keeps its own: an ordinary object whose surfaces
  have the `materials\occ` game material, which the Build exports to `level.hom` and leaves out of the geometry.
  `level.som`, the sound occluders, comes back as `<name>_som`, flagged as a sound occluder. Each surface takes the game
  material whose sound occlusion factor is its polygons', which needs the game material library.
- The Build cuts ladders and breakable glass or boards out of the geometry and exports them as entities, so they come
  back as objects and leave the spawn points. A ladder becomes `<name>_ladder_<N>`: the box the Build made of it, less
  its back face, standing at its foot, of the entity's game material or `materials\fake_ladders`. A breakable part
  becomes `<name>_breakable_<N>` from its `meshes\brkbl#<n>.ogf`, its fragments joined again, where its entity placed
  it. A rebuild exports both again.
- A sector that draws only trees would have no object to name. On such a level the sectors and portals are left out, so
  a rebuild does not number its sectors differently, and the sidecar names each empty sector.
- The objects go under `--objects` at `levels\<name>\`, each with its own `.object.xrf.json` sidecar. By default that is
  the `objects` directory beside the destination's directory: `rawdata/objects` for a scene in `rawdata/levels`, as the
  SDK lays them out.
- A tree model whose impostor the level's `level_lods.dds` holds gets its strip back as `lod\lod_<model>.dds` and
  `_nm.dds` under `--textures`, the textures the editor's Build loads for it. By default that is `gamedata/textures`
  beside the destination's `rawdata` directory: `editors/gamedata/textures` for a scene in `editors/rawdata/levels`.
  `level build` merges them back into `level_lods.dds` from the root its own `--textures` names.
- Surfaces keep their texture and engine shader, and by default (`--surfaces infer`) take a game material and a compiler
  shader inferred from the level. Each triangle takes the game material of the collision face in `level.cform` it lies
  on; a surface whose triangles lie on different materials, or on none, is split. The compiler shader is one in
  `shaders_xrlc.xr` that fits what the level shows: whether the surface collides, and whether it is lit per vertex or by
  lightmap. Among those that fit, `--library <objects root>` (repeatable, such as an SDK's `rawdata/objects`) prefers
  the one its objects pair most with the surface's engine shader.
- The two libraries are `gamemtl.xr` and `shaders_xrlc.xr` in the level's `gamedata`, or `--game-materials` and
  `--compiler-shaders`. `--surfaces defaults` gives every surface the editor's `default` instead. The sidecar's
  `geometry.surfaces` records the libraries, how many triangles lay on a collision face, were traced to one or lay on
  none, and each compiler shader chosen with why.

Entities come from `level.spawn` and `level_rs.spawn`, each as a spawn point at its place with its packet as spawn data,
and a zone's or restrictor's volume as a shape attached to it. An entity whose class the decompile cannot read, such as
a multiplayer item or a Clear Sky or Shadow of Chernobyl packet, is left out: the command warns, and the sidecar names
each one with its reason.

A game starts from its root spawn, `spawns/all.spawn`, and mods often edit it after it was built, so its objects on the
level replace `level.spawn`'s where the game tree has one:

- `--spawn <name>` reads `spawns/<name>.spawn` instead, such as a test spawn built for one level. It must exist.
- Without `--spawn`, `all.spawn` is read where it exists, else the entities come from `level.spawn` alone.
- The ids the AI compiler assigns while building the root spawn (game and level vertices, a level changer's destination)
  are taken from `level.spawn`, as the editor wrote them, so a scene built again gives the same `level.spawn`.
- The sidecar's `rootSpawn` lists every object the two files disagree on, and keeps the update packets a rebuilt root
  spawn would not reproduce. The command reports the counts.

- It writes `<name>.level`, a directory `<name>` beside it holding one `.part` file per tool, and a
  `<name>.level.xrf.json` sidecar. The name comes from `--dest`.
- The sidecar lists each source file with its SHA-256, each written file and its sources, and what the decompile chose
  rather than read (object names, level options, the camera, build parameters). The scene's GUID is derived from the
  sources, so decompiling the same level twice gives the same scene.
- An existing `.level`, part directory, sidecar or object is refused unless `--force` is given. Even with `--force`, a
  part directory holding a part the decompile would not write is refused, since the editor would read that part as
  belonging to the scene. A refusal writes nothing.

## Build

`level build` goes the other way for the same files. It reads a scene's `.level` and parts and writes into a level
directory what the Level Editor's Build writes from those parts:

- `level.game`, `level.env_mod` and `level.fog_vol` every time;
- `level.spawn`, and `level_rs.spawn` for entities marked to respawn in multiplayer, where the scene places entities;
- `level.ps_static`, `level.snd_static`, `level.snd_env` and `level.wallmarks` where the scene holds their objects.

Each entity takes its name, position and angles from its spawn point, and its volume from the attached shape, as the
editor's Build does. It does not write geometry or `build.prj` yet.

```powershell
xrf-cli level build --path ./rawdata/levels/zaton.level --dest ./gamedata/levels/zaton --force
```

- A file it would write that is already there is refused unless `--force` is given. The report says of each file whether
  it is new, the same bytes as before, or changed. A level decompiled and built back into its own directory reports
  every file the same but, on most levels, `level.spawn`: the scene saves floats as text with six decimals, as the
  editor does, which rounds the few that held more, an entity the decompile left out is missing, and so are ladders and
  breakables, which the scene holds as geometry that `level build` does not export yet.
- The editor's Build moves every file it may write aside as a `.~` backup first, so a file the scene no longer produces
  goes away. `level build` moves and deletes nothing. It refuses, naming the file, when the directory holds one the
  scene does not build, such as a `level.ps_static` from a scene that now places no particles.

## Albedo

`level albedo` takes the base texture of every surface in the shader table that has a colour of its own: opaque and
alpha-tested surfaces, not glass, glows, water or anything else blended or self-lit. It finds each texture as the
renderer does, in the level's own directory first and then the shared `textures` tree, and measures it once however many
surfaces draw it. A cut-out surface is measured over the texels its alpha test keeps.

Each texture is decoded to linear light as the renderer decodes it and compared with the range real, unlit materials
span: 0.03 (charcoal) to 0.9 (fresh snow). Its verdict, `dark`, `within` or `bright`, judges the median texel, so a
highlight or a black border does not decide it. The report also gives the 5th and 95th percentile, the shares of texels
past either bound, and the coarse spread: how many stops apart the brighter and darker parts of the texture sit, which
light and shade painted into the texture widen (an atlas of unlike materials widens it too).

It is a report rather than a check: textures with painted light are the game's art as shipped, so outliers are counted
and never fail the run. A texture no root holds is reported as `missing`. The command needs `--root` and `--level`,
because a level directory alone holds no textures to resolve. Detail textures laid over a base, and the textures of
models placed by the spawn, are not in the shader table, so they are not measured.

## Materials

A game material, from `gamemtl.xr`, is what a surface is made of to the game: the sound a footstep makes on it, how a
bullet passes through it, whether grass grows on it. The compiled level stores it only on the collision form, never on
the geometry that is drawn. `level materials` finds it for every render surface by looking where each of its triangles
lies: of the collision faces a short line through the triangle's centre crosses, the nearest is the one built from it,
and the triangle's area is counted against that face's material. Taking the nearest, rather than the first a ray from
above meets, keeps a floor's material where a plate or grating lies a few centimetres over it.

```powershell
xrf-cli level materials --root ./stalker-anomaly --level zaton --report zaton-materials.json
```

```text
Reading the materials beneath compiled level levels\zaton
917 surfaces over 19358587 square metres, 52% of it on collision faces of 24 materials
```

Exit code: `0`.

Each surface in the report, one per shader table entry, lists its shader and textures, its area, the share of it lying
on no collision face, and the materials beneath the rest by their share. Foliage, grass and the landscape past the
playable edge have no collision, so they read as uncovered; a surface the level's author never assigned a material reads
as `default`. The materials are named from the installation's `gamemtl.xr`; a level read by `--path` alone reports their
ids. The command needs the level's `level.cform` beside `level` and `level.geom`.

{{#include reference/level.md:commands}}
