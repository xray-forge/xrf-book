# Level CLI

Level commands work on a compiled level, the files the game loads from `levels\<name>`: the `level` bundle with its
shader table, sectors, portals, lights and visuals, the `level.geom` and `level.geomx` buffers the visuals draw from,
and companions such as `level.cform` (collision), `level.ai` (the AI map) and `level.spawn`. They do two kinds of work:

- `level info`, `level verify`, `level albedo` and `level materials` read a level and report on it.
- `level decompile`, `level build` and `level compile` take a level apart into a Level Editor scene and make it again.
  [Decompile](#decompile) turns a level directory into a scene; [build](#build) writes the scene's game files, and the
  inputs the compilers need, into a level directory; [compile](#compile) turns those inputs into the level's geometry,
  light, details and AI map.

The reading commands name the level two ways: `--path` is a level directory on disk, and `--root` with `--level` is one
of an installation's levels, read through the game's file system by its engine name `levels\<name>`. A stock Anomaly
installation ships its levels inside `.db` volumes, with no directory to point at, so the second way is the one that
reads a real game. `--root` without `--level` fails and names the levels the roots hold. The decompile, build and
compile commands work on directories on disk: each takes `--path`, and the first two a destination, `--dest`.

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

It also checks the AI map, `level.ai`, as the SDK's `xrAI -verify` does. A node linked to itself fails the level. A link
with no link back, and a part of the map no link reaches, are warnings that leave it passing: GSC shipped every
single-player level with one-way links. The warnings give their counts and the first nodes' positions; the JSON report
counts the map in `aiCensus`. A level without `level.ai`, as a multiplayer one may be, skips these checks.

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
  as one object, welded, with mirrored triangles folded into two-sided faces.
- Given `--library <objects root>` (repeatable, such as an SDK's `rawdata/objects`), the library's objects the level
  holds are found in that geometry and come back as scene objects placing them by reference, at their place, turn and
  scale, their triangles taken out of the sector objects; a tree model the library holds is placed by its library name
  too. Each object found is copied, with its thumbnail, from the library to `--objects`, so the scene opens without the
  library. The library's objects in `levels\<name>\`, what an earlier decompile of this level wrote, are not looked for.
  The sidecar's `geometry.instances` lists each copy with its hash and each placement with how well it fits. Geometry no
  library object matches stays in the sector objects. A library file that does not read is warned of and listed among
  the refusals with its path and why; a link back to a folder above it, or to nothing, is listed there too, not
  followed. An object reference or texture name with `..` or a drive in it refuses the decompile before anything is
  written.
- Trees come back as multi-use objects, `<name>_tree_<N>`, one per model, with a scene object per placement.
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
  lightmap. Among those that fit, the `--library` roots prefer the one their objects pair most with the surface's engine
  shader.
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
- An existing `.level`, sidecar, file in the part directory or object of the level is refused unless `--force` is given,
  before anything is decompiled, so a refusal writes nothing. With `--force`, the decompile replaces them and removes
  the parts and objects of this level it no longer writes, since the editor would read a part left behind as belonging
  to the scene. The report lists what it removed.

## Build

`level build` goes the other way for the same files. It reads a scene's `.level` and parts and writes into a level
directory what the Level Editor's Build writes from those parts:

- `level.game`, `level.env_mod` and `level.fog_vol` every time;
- `level.spawn`, and `level_rs.spawn` for entities marked to respawn in multiplayer, where the scene places entities;
- `level.ps_static`, `level.snd_static`, `level.snd_env` and `level.wallmarks` where the scene holds their objects.

Each entity takes its name, position and angles from its spawn point, and its volume from the attached shape, as the
editor's Build does.

Given the game material library (`--game-materials`), it also writes what `level compile` takes, as the SDK's compilers
took it: `build.prj` with the build options the scene sets, `level.ltx`, the occluders (`level.hom`, `level.som`),
`build.details` and its atlas, `build.aimap`, the tree impostors' atlas (`level_lods.dds`, `level_lods_nm.dds`) and each
breakable part (`meshes/brkbl#<n>.ogf`). It reads the scene's objects from the objects roots `--objects` names,
repeatable, or by default from `objects` beside the scene's `levels` directory, where `level decompile` writes them.
Details and impostors read their textures from `--textures`, by default `gamedata/textures` beside the scene's `rawdata`
directory, where `level decompile` writes them, so a decompiled scene builds back without naming it. The ladders and
breakables it cuts out of the geometry open the spawn list, as the editor's Build places them.

```powershell
xrf-cli level build --path ./rawdata/levels/zaton.level --dest ./gamedata/levels/zaton --objects ./rawdata/objects --objects ./sdk/rawdata/objects --game-materials ./gamedata/gamemtl.xr --force
```

- The build options in `build.prj` are the scene's, as the Level Editor writes them. A quick look at a level comes from
  `level compile --quality draft` instead ([Light quality](#light-quality)).
- The impostor atlas grows to 4096 a side, as Anomaly's largest levels ship it. `--sdk-impostor-atlas` keeps SDK 0.7's
  limit of 2048, which refuses a level of more than 128 tree impostors as that Level Editor does.

- Any file it may write that is already there, names compared without case, is refused unless `--force` is given, before
  the scene is built, so a refusal writes nothing. The report says of each file whether it is new, the same bytes as
  before, or changed. A level decompiled and built back into its own directory reports every file the same but, on most
  levels, `level.spawn`: the scene saves floats as text with six decimals, as the editor does, which rounds the few that
  held more, and an entity the decompile left out is missing.
- The editor's Build moves every file it may write aside as a `.~` backup first, so a file the scene no longer produces
  goes away. With `--force`, `level build` removes such a file instead, such as a `level.ps_static` from a scene that
  now places no particles or a breakable part it no longer cuts, and the report lists what it removed.
- A scene that places objects is refused without `--game-materials`, since only the compiler's files carry them.
- A scene the Level Editor would load as something else is refused, naming what: a part saved for another scene (another
  GUID), a group member past the group's `ingroup_count` or missing under it, or a way point linking to itself. GSC's
  four SDK scenes hold none of these.

## Compile

`level compile` takes the level directory `level build` wrote and compiles it into the level the game loads. XRF's own
compilers do the work, in the command's own process, so it needs no SDK. It works in four steps:

| Step       | Reads                                            | Writes                                                                                          |
| ---------- | ------------------------------------------------ | ----------------------------------------------------------------------------------------------- |
| `geometry` | `build.prj`                                      | `level`, `level.geom`, `level.geomx`, `level.cform`, `build.cform`, `build.lights`, lightmaps   |
| `light`    | `build.prj`, in the geometry's pass              | the light of the vertices, trees and lightmaps, and of textures lit implicitly, such as terrain |
| `details`  | `build.details`, `build.cform`, `build.lights`   | `level.details`, lit                                                                            |
| `ai`       | `build.aimap`, and `build.cform` for full covers | `level.ai`                                                                                      |

Every step runs by default, in that order whatever the order given; `--steps` names some of them. The light is baked in
the geometry's pass, since the lightmap pages decide how the surfaces join into visuals, so `light` runs only with
`geometry`. `geometry` alone writes the level with neutral light, the quick way to look at a change to its shapes.

```powershell
xrf-cli level compile --path ./gamedata/levels/mp_darkvalley
xrf-cli level compile --path ./gamedata/levels/mp_darkvalley --steps geometry
xrf-cli level compile --path ./gamedata/levels/mp_darkvalley --steps geometry,light,details --quality draft
```

It prints each step's time, with what the geometry compiler made, what the light baker lit and where it traced, the
slots lit for the details, and the nodes and graph for the AI map, then the level.

- `--gamedata` names where the compilers read the game materials (`gamemtl.xr`), the compiler shaders
  (`shaders_xrlc.xr`) and the textures: by default the gamedata holding the level. Every texture `build.prj` names needs
  its `.thm` there or under a textures root `--textures` names, such as the one `level decompile` wrote impostor strips
  into. It repeats, each file read from the first root holding it and the gamedata's `textures` last.
- `--quality` sets how finely the light step lights, `scene` by default or `draft` for a cheap estimate
  ([Light quality](#light-quality)).
- `--backend` picks what traces the light, details and AI map rays: the GPU where it can, else the CPU
  ([Ray tracing](#ray-tracing)).
- Before any step runs, it checks the inputs in the table above, the compiler shader library and every texture's `.thm`
  where a step reads them, so a missing file stops it at once rather than minutes into a step. What the geometry step
  writes counts as there for the steps after it.
- Any file a step it runs writes that is already there, lightmaps and the implicitly lit textures included, refuses the
  compile before any step runs unless given `-f/--force`. With it, the geometry step also removes the lightmap pages of
  an earlier compile it no longer writes.

The times below were measured on a 16-core machine with an RTX 5080. Where a time for SDK 0.7's compilers (xrLC,
xrDO_Light, xrAI) is given, it is a historical comparison from runs made before XRF's compilers replaced them;
`level compile` does not run them.

### Light quality

`--quality` sets the settings the light step lights by:

| Quality | Lights by                                                                                                                                                    |
| ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `scene` | `build.prj`'s own sky and sun, lightmap density and samples, as the Level Editor built them; the default                                                     |
| `draft` | the sky and sun the Level Editor builds at quality 1 (26 and 9 lights), lightmaps of 3 texels a metre and one sample each, and each tree lit from one sample |

`draft` regenerates the sky and sun from the level's sun direction exactly as the Level Editor would, and keeps the
lamps placed under the sky's layer. A level already built at quality 1 compiles the same either way. The geometry step
writes `build.lights` with the lights the light step lights by, so the details step lights by them too.

For a full-quality bake, set the build options in the scene in the Level Editor, as for the SDK's own Build;
`level build` writes them into `build.prj` as they are.

`--quality` is not the scene's build quality, the draft or high setting `build.prj` also carries. That one decides, as
it does for xrLC, whether visuals are made progressive and whether textures are lit implicitly; `--quality` changes
neither.

### Ray tracing

`--backend` picks what traces the rays of the light, details and AI map steps:

| Backend | Traces                                                                 |
| ------- | ---------------------------------------------------------------------- |
| `auto`  | on the GPU where one traces rays, on the CPU otherwise; the default    |
| `gpu`   | with the GPU's ray tracing alone; refused where no GPU traces rays     |
| `cpu`   | on the CPU's workers; the reference the SDK's runs are matched against |

`level compile` and `level decompile` take `-j` to bound how many cores they use, `auto` (every one) by default.

The GPU traces on Vulkan, or on D3D12 where DXC's `dxc/dxcompiler.dll` stands beside the CLI; either needs a GPU with
ray tracing. A CLI built with `--no-default-features` has no GPU backend and traces on the CPU. How closely the GPU's
results follow the CPU's is given with the light and details steps.

### Geometry

The geometry step compiles `build.prj` as xrLC does, phase by phase: the weld and the faces it refuses, the cut of edges
12.8 m or longer, normals, the collision form (`level.cform`) and the compilers' ray model (`build.cform`), the tangent
basis, the split into visuals by material and size, multi-use models placed as trees with their LODs, each sector's
hierarchy and its portals, and `build.lights` from the project's light layers. Above draft build quality a visual of 128
faces or more is made progressive, its detail stepping down by sliding windows, as xrLC's are.

Against SDK 0.7's own runs on the same `build.prj`, every visual triangle, sector and portal is the same, with normals
and coordinates within their stored precision; the collision forms hold the same faces but for the few where xrLC's weld
met corners in another order, as two of its own runs differ.

The collision form is not simplified, as xrLC simplifies it above draft build quality. The game collides with every
face.

Visuals split a little differently from xrLC's, as they split by lightmap page and our charts sit on their pages in
other places. Without the light step every lightmapped surface samples one small pair of neutral light (an open sky for
the sun and half the hemisphere, no static light), the renderers lighting the rest in real time, and more surfaces join
into each visual.

### Light

The light step lights the level as xrLC does, from `build.prj`'s lamps, sun and sky. Vertices take each one's sky term,
a vertex-lit surface's colour and sun, and the trees' light, within a step of xrLC's on nearly every vertex. Lightmapped
surfaces are cut into charts as xrLC cuts them, over 99.5% of them the same faces; each chart is lit texel by texel,
compressed where its light allows as xrLC compresses it, and packed onto lightmap pages, as many as xrLC's. Compared
with xrLC's at points across every lightmapped surface, the light differs as little as two of xrLC's own bakes of a
level do. A texture xrLC lights implicitly, such as terrain's, takes its light over its own coordinates as xrLC's does,
written beside the level as `<texture>.dds` and `<texture>_lm.dds`; at draft build quality, as xrLC, the step lights
none.

On the GPU ([Ray tracing](#ray-tracing)) the light is the CPU's on all but a few points in a thousand, a few steps
apart; a chart compressed by its light can carry that across it, so pages differ from the CPU's by up to a few steps
here and there, and from xrLC's as little as the CPU's do. The GPU pays most where lamps are many, as indoors. The
geometry and light steps together with `--quality draft`:

| Level           | CPU  | GPU  |
| --------------- | ---- | ---- |
| `l01_test`      | 43 s | 3 s  |
| `labx8`         | 35 s | 12 s |
| `mp_darkvalley` | 4 s  | 4 s  |
| `jupiter`       | 73 s | 69 s |

For historical comparison, SDK 0.7's xrLC took 28 s on `mp_darkvalley` at draft build quality, and about 15 min on
`jupiter` at the light settings `draft` uses, 11 of them placing its charts.

### Details

The details step lights every planted slot of `build.details` as xrDO_Light does: the sun, the sky and the level's
lights traced to 225 points across each 2 m slot over the compilers' ray model (`build.cform`), a ray dimmed by the
alpha of a texture it crosses and passed by a surface that casts no shadow. It writes the five light values a slot
stores, the grass's sun and sky shade among them.

Against SDK 0.7's own runs on the same files it lights all but a handful of slots exactly, each of those one step of
sixteen off. xrDO_Light remembers the last surface that shadowed each light and tests it first, a shortcut that changes
its result with the order and the threads it lights in; XRF traces every ray in full, so a run gives the same file on
any machine.

On the GPU ([Ray tracing](#ray-tracing)) the light is the CPU's but for a few slots in a hundred thousand, each one step
off. The details step alone, with the last column a historical comparison:

| Level, lights                  | CPU   | GPU   | SDK 0.7's xrDO_Light (historical) |
| ------------------------------ | ----- | ----- | --------------------------------- |
| `mp_darkvalley`, default       | 14 s  | 3.5 s | about 12 min                      |
| `jupiter`, at `draft` settings | 71 s  | 20 s  | about 23 min                      |
| `jupiter`, as shipped          | 406 s | 51 s  | not run                           |

### AI map

The AI step compiles the nodes the Level Editor exported (`build.aimap`) into `level.ai` as SDK 0.7's `xrAI -f` does:
each node's cell, height and plane packed over the map's box, the nodes sorted by cell with their links renumbered. Its
covers, how open each node is in four directions from a standing and a crouching height, are traced against the
compilers' ray model (`build.cform`) as xrAI traces them: from each node at a corner of the map toward every node it
reaches within 30 m, a ray dimmed by the alpha of a texture it crosses; every other node then takes a mix of the corners
it walks straight to. `--backend` picks what traces those rays ([Ray tracing](#ray-tracing)).

- `--ai-covers none` leaves every cover 0, as xrAI's `-draft` does, and needs no `build.cform`, so it runs on what
  `level build` wrote before the geometry is compiled. `full`, the default, needs `build.cform`, `build.prj` and
  `shaders_xrlc.xr`.
- A `level.ai` replaced with `--force` keeps its identity while the new map's nodes have the same cells and links, so
  the game's spawn, built against the old map, still loads the level. A map that changed takes an identity hashed from
  its nodes, the same for the same input; it needs a new spawn ([Spawn](#spawn)).
- Before it writes, the map is checked as `level verify` checks one: a node linked to itself stops the step with the
  file left as it was, and links with no link back and parts no link joins are warnings with their counts.

Against SDK 0.7's own runs on the same files, everything but the covers is xrAI's byte for byte, and every cover is
within one step of sixteen, all but a few in ten thousand exact. Like the details, XRF traces every ray in full rather
than test the surface that blocked a ray before, so a run gives the same file on any machine. `jupiter`'s 1.5 million
nodes take 35 s on the CPU and 10 s on the GPU; for historical comparison, SDK 0.7's xrAI took about 7 min.

### Spawn

The spawn is not a step: `all.spawn` is built from every level of a game at once, as SDK 0.7's `xrAI -s` builds it, by
[`spawn build`](spawn.md).

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
