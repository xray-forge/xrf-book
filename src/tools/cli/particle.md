# Particle CLI

Particle commands inspect, verify, pack, unpack, and round-trip `particles.xr` data.

## Examples

```powershell
xrf-cli particle info --path ./particles.xr
xrf-cli particle unpack --path ./particles.xr --dest ./particles_unpacked --force
xrf-cli particle pack --path ./particles_unpacked --dest ./particles.xr --force
xrf-cli particle repack --path ./particles.xr --dest ./particles.repacked.xr
xrf-cli particle re-unpack --path ./particles_unpacked --dest ./particles_unpacked_roundtrip
xrf-cli particle verify --path ./particles.xr
xrf-cli particle verify --path ./particles_unpacked --unpacked
```

`particle repack` reads a packed file and writes another packed file; `particle re-unpack` does the same between two
unpacked folders. Both exist to prove a round trip preserves the data, which is what makes an editing workflow safe.

Packing and unpacking are lossless without being byte-identical, so comparing a repacked file to its source by hash
reports a difference that is not a defect.

## Failure notes

Packing fails if the output file already exists and `--force` is not supplied. Unpacking fails if the destination folder
already exists and `--force` is not supplied.

## Command reference

{{#include reference/particle.md:commands}}
