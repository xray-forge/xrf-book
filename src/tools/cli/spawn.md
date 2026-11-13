# Spawn CLI

Spawn commands build, inspect and convert ALife `.spawn` files. Build a root spawn from the levels, or unpack one for
editing, then rebuild and verify a separate output before replacing the game's spawn file.

## Build a root spawn

`build` does what SDK 0.7's `xrAI -s` does: it reads each level's `level.ai`, `level.spawn` and `level.game` from a
gamedata root and writes one spawn holding the game graph, its cross tables, every entity, the anomalies' artefact
points and the patrols. Run it after a change to a level's AI map, entities, graph points, level changers or patrols;
geometry, lighting and details do not need it.

```powershell
xrf-cli spawn build --path ./gamedata --out test_jupiter
```

The root holds `configs/game_levels.ltx` and the levels' directories under `levels/`. Without `--levels`, every level
`[levels]` lists whose directory holds the three files is built; `--levels zaton,jupiter` builds those alone. The spawn
is written as `spawns/<out>.spawn`. `all` is the game's own spawn, and writing over it needs `--force`.

Example output excerpt — vanilla's five levels:

```text
zaton: Restrictor zat_surge_hide_b40 cuts 69 AI map nodes off, the first node 1495511 at (313.6, 41.7, -389.9)
Built ./gamedata/spawns/test_jupiter.spawn from zaton, jupiter, pripyat, labx8, jupiter_underground: 6464 objects,
39584 artefact points, 4634 patrols, 153 warnings
```

Exit code: `0`.

Built from vanilla's own levels, the spawn matches the shipped `all.spawn`: the game graph, every entity, every artefact
point and every patrol point with an AI node under it. What differs is what xrAI leaves to chance: the GUIDs, the
patrols' order, the game vertex of a patrol point with no node under it, and fields xrAI never sets. The same levels
always give the same file, with the same GUID, so a rebuild that changes nothing keeps saved games loading; any real
change gives a new GUID, and old saves of that spawn no longer load.

`--start <level>` puts the actor on another level, at the first game vertex of that level a search from its own reaches,
as xrAI's `-start` does. Naming the actor's own level leaves it where it stands.

The build warns, and goes on, where xrAI lets something through:

- a graph point left out of the game graph, too close to another, off the AI map, or on another's node;
- a graph point connecting to a level outside the build, whose connection is left out;
- a restrictor flagged for the separator check whose border cuts AI nodes off the rest of the map, with the count and
  where the first one is. `--strict-separators` refuses the build instead, as xrAI does without `-no_separator_check`.
  Vanilla's levels give 153 such warnings, which is why GSC built them without the check.

It refuses, with exit code `1`, what xrAI asserts on: no actor or more than one, a level with no entity or no graph
point, two entities sharing a story id, a level changer leading to a level not built or to a graph point its level
lacks, two patrols of one name, two neighbouring graph points no path on the AI map joins, and an unknown start level.

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

`info` reports the header and counts of objects, artifact spawns, patrols, and graph entries. It also reads a level's
`level.spawn`, the list of objects a level spawns, telling the two kinds apart by the file's contents rather than its
name:

```powershell
xrf-cli spawn info --path ./levels/jupiter/level.spawn
```

Example output excerpt — inspect a level's spawn list:

```text
Level spawn list information:
Objects count: 2830
Sections count: 175
Section 'af_oasis_heart' objects: 1
Section 'ammo_11.43x23_fmj' objects: 12
```

A level's list has no header, so `info` counts its objects by the config section each is built from. The `--report`
result says which kind was read in its `kind` field, `root` or `level`. A file that is neither is refused with exit code
`1`.

After editing the exported representation of `all.spawn`:

```powershell
xrf-cli spawn pack --path ./all_spawn --dest ./all.rebuilt.spawn
xrf-cli spawn verify --path ./all.rebuilt.spawn
```

Verification reads the packed file and checks that its structure can be parsed. Use [gamedata verification](gamedata.md)
for checks involving the surrounding assets; test the affected spawning behavior in game.

`verify` and `repack` also take a level's `level.spawn`. Verification of a level's list checks that every object opens
as a spawn packet with its section, name and position; the rest of each packet is carried as it is, and repacking writes
it back byte for byte. A level's list has no unpacked form yet: `unpack` refuses one with exit code `1`, and `pack`,
`re-unpack` and `verify --unpacked` read only a root spawn's folder.

## Re-serialize a spawn file

```powershell
xrf-cli spawn repack --path ./all.spawn --dest ./all.repacked.spawn
```

`repack` reads a packed file and writes another packed file. It does not automatically compare the output with the
source or prove that an editing workflow preserved behavior.

## Failure notes

Every command that writes requires `--dest`. Packing and unpacking reject existing destinations unless `--force` is
supplied. Choose a new destination while reviewing edits; use `--force` when replacing that output is intended.

If the source cannot be parsed, inspect the reported chunk or format error before editing. Repacking requires a readable
source and does not repair truncated data.

## Command reference

{{#include reference/spawn.md:commands}}
