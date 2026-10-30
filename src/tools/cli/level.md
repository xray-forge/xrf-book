# Level CLI

Level commands read one compiled level: the `level` bundle with its shader table, sectors, portals, lights and visuals,
and the `level.geom` and `level.geomx` buffers the visuals draw from. Every command names the level the same two ways:
`--path` is a level directory on disk, and `--root` with `--level` is one of an installation's levels, read through the
game's file system by its engine name `levels\<name>`. A stock Anomaly installation ships its levels inside `.db`
volumes, with no directory to point at, so the second way is the one that reads a real game. `--root` without `--level`
fails and names the levels the roots hold.

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
lies: a short ray through the triangle's centre meets the collision face built from it, and the triangle's area is
counted against that face's material.

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
