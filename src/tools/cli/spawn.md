# Spawn CLI

Spawn commands inspect and convert ALife `.spawn` files. Unpack a file for editing, then rebuild and verify a separate
output before replacing the game's spawn file.

## Examples

Run from a working directory containing `all.spawn`, with new output paths:

```powershell
xrf-cli spawn info --path ./all.spawn
xrf-cli spawn unpack --path ./all.spawn --dest ./all_spawn
```

Example output excerpt — inspect spawn data:

```text
Spawn file information:
Version: 10
GUID: a117405b-0a2e-a781-4ab5-f7aa88ae759c
Levels count: 5
Objects count: 262
Artefact spawn points: 256
Patrols: 4636
Level version: 10
Level graph vertices: 934
Level graph points: 512
Level graph edges: 2568
```

Exit code: `0`.

`info` reports the header and counts of objects, artifact spawns, patrols, and graph entries. After editing the exported
representation:

```powershell
xrf-cli spawn pack --path ./all_spawn --dest ./all.rebuilt.spawn
xrf-cli spawn verify --path ./all.rebuilt.spawn
```

Verification reads the packed file and checks that its structure can be parsed. Use [gamedata verification](gamedata.md)
for checks involving the surrounding assets; test the affected spawning behavior in game.

## Re-serialize a spawn file

```powershell
xrf-cli spawn repack --path ./all.spawn --dest ./all.repacked.spawn
```

`repack` reads a packed file and writes another packed file. It does not automatically compare the output with the
source or prove that an editing workflow preserved behavior.

## Failure notes

Packing and unpacking reject existing destinations unless `--force` is supplied. Choose a new destination while
reviewing edits; use `--force` when replacing that output is intended.

If the source cannot be parsed, inspect the reported chunk or format error before editing. Repacking requires a readable
source and does not repair truncated data.

## Command reference

{{#include reference/spawn.md:commands}}
