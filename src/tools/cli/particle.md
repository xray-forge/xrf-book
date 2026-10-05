# Particle CLI

Particle commands inspect and convert `particles.xr` libraries. Use the unpacked representation to edit a library, then
verify the packed result before adding it to gamedata.

## Examples

Run from a working directory containing the source `particles.xr`. Use new destinations for the unpacked library and
rebuilt file:

```powershell
xrf-cli particle info --path ./particles.xr
xrf-cli particle unpack --path ./particles.xr --dest ./particles_unpacked
```

Example output — inspect particles:

```text
Read particle file ./gamedata/particles.xr
Particles file information:
Version: 1
Effects count: 921
Groups count: 350
```

Exit code: `0`.

Edit the exported files, then validate and pack them:

```powershell
xrf-cli particle verify --path ./particles_unpacked --unpacked
xrf-cli particle pack --path ./particles_unpacked --dest ./particles.rebuilt.xr
xrf-cli particle verify --path ./particles.rebuilt.xr
```

`verify` checks that the selected representation can be read. It does not resolve the library's texture dependencies.
After installing the rebuilt file, use [gamedata verification](gamedata.md) to check the library in its asset context,
and inspect the affected effects in game.

## Look up effects and groups

```powershell
xrf-cli particle list --path ./particles.xr --kind group --name campfire
xrf-cli particle find --path ./particles.xr --texture pfx/pfx_splash_02 --effect explosions\effects\campfire_sparks
xrf-cli particle inspect --path ./particles.xr explosions\campfire_boar_grill
```

`list` names every effect and group, or those of one kind or whose name holds a text: an effect with its shader,
textures, particle limit and action types, a group with its effects and the children they spawn. `find` answers which
effects sample a texture or draw with a shader or run an action type, and which groups play an effect or spawn it as a
child, each match with every way it matched. A texture matches in any case, with either slash and with or without its
extension, as the engine finds it. `inspect` explains one effect (sprite, flags, frames, collision, actions) or group
(each effect's time window, flags and children), and reports it whole with `--json` or `--report`. A name nothing holds
fails with exit code `1`, naming the names that contain it.

Example output — explain a group:

```text
group explosions\campfire_boar_grill
  time limit: none, loops
  effects: 5
    0. explosions\effects\campfire_boar_grill_smoke from 0 s to 0 s (Enabled)
    1. explosions\effects\campfire_boar_grill_sparks from 0.5 s to 0 s (Enabled)
    2. explosions\effects\fire_distort from 0 s to 0 s (Enabled)
    3. explosions\effects\campfire_boar_grill_vacuum from 0 s to 0 s (Enabled)
    4. explosions\effects\campfire_boar_grill_drop from 0 s to 0 s (Enabled, OnBirthChild, OnDeadChild)
       on birth: explosions\effects\campfire_boar_grill_drop_smoke
       on death: explosions\effects\campfire_boar_grill_drop_sparks
```

Exit code: `0`.

## Re-serialize a library

```powershell
xrf-cli particle repack --path ./particles.xr --dest ./particles.repacked.xr
xrf-cli particle re-unpack --path ./particles_unpacked --dest ./particles_unpacked_roundtrip
```

`repack` reads a packed library and writes another packed file. `re-unpack` imports an unpacked library and exports it
to another directory. Neither command automatically compares the result with its source. Serialization can change bytes,
so a hash difference alone does not demonstrate a change to the particle definitions.

## Failure notes

Packing rejects an existing output file, and unpacking rejects an existing destination directory, unless `--force` is
supplied. Use that flag only when replacing the selected output is intended.

A successful read or conversion establishes that the tool accepts the data. It does not establish visual equivalence or
correct behavior of every effect.

## Command reference

{{#include reference/particle.md:commands}}
