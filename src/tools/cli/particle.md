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
