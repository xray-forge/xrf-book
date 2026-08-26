# Spawn CLI

Spawn commands inspect, verify, pack, unpack, and round-trip ALife `.spawn` files.

## Examples

```powershell
xrf-cli spawn info --path ./all.spawn
xrf-cli spawn unpack --path ./all.spawn --dest ./all_spawn --force
xrf-cli spawn pack --path ./all_spawn --dest ./all.spawn --force
xrf-cli spawn repack --path ./all.spawn --dest ./all.repacked.spawn
xrf-cli spawn verify --path ./all.spawn
```

`spawn info` reports the header, object, artefact spawn, patrol, and graph counts. `spawn repack` reads a packed file
and writes another packed file, which is how a round trip is shown to preserve the data.

## Failure notes

Packing and unpacking reject existing destinations unless `--force` is supplied.

## Command reference

{{#include reference/spawn.md:commands}}
