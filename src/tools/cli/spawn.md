# Spawn CLI

Spawn commands build, inspect and convert ALife `.spawn` files. Build a root spawn from the levels, or unpack one for
editing, then rebuild and verify a separate output before replacing the game's spawn file.

## Build a root spawn

`build` does what SDK 0.7's `xrAI -s` does: it reads each level's `level.ai`, `level.spawn` and `level.game` from a
gamedata root and writes one spawn holding the game graph, its cross tables, every entity, the anomalies' artefact
points and the patrols. Run it after a change to a level's AI map, entities, graph points, level changers or patrols;
geometry, lighting and details do not need it.

```powershell
xrf-cli spawn build --gamedata ./gamedata --dest ./gamedata/spawns/test_jupiter.spawn
```

The root holds `configs/game_levels.ltx` and, by default, the levels' directories under `levels/`. Without `--levels`,
every level `[levels]` lists whose directory holds the three files is built; `--levels zaton,jupiter` builds those
alone. `--dest` names the `.spawn` file to write; without it the build writes `spawns/all.spawn`, the game's own.
Writing over a spawn already there, the name compared without case, needs `--force`.

Levels built outside the gamedata root, such as into a working copy by `level build` and `level compile`, are read from
the directory `--levels-root` names; the level list still comes from the root:

```powershell
xrf-cli spawn build --gamedata ./gamedata --levels-root ./work/levels --levels zaton,jupiter --dest ./work/spawns/test.spawn
```

Like the other heavy commands, `build` takes `-j` to bound how many cores it uses.

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
it back byte for byte.

## Unpack a level's spawn list

`unpack` also takes a level's `level.spawn`. Its folder holds one file, `level_spawns.ltx`, with a section per object in
the list's order, named by its place:

```powershell
xrf-cli spawn unpack --path ./levels/l04u_labx18/level.spawn --dest ./labx18_spawn
xrf-cli spawn pack --path ./labx18_spawn --dest ./level.rebuilt.spawn
```

Example output excerpt — a list holding objects of Shadow of Chernobyl's layout:

```text
Unpacking ./levels/l04u_labx18/level.spawn into ./labx18_spawn
Wrote 0 objects typed and 579 verbatim
  579 written verbatim: Assertion error: Spawn packet is of spawn version 118, Shadow of Chernobyl's, whose layout is not supported
Unpacked ./labx18_spawn in 4 ms
```

Exit code: `0`.

An object the tools read is written with the same keys an object of an unpacked `all.spawn` has. An object they cannot
read, such as one saved in Shadow of Chernobyl's layout, is written verbatim: its section, name, position and direction
as keys, the rest of its packet as base64 under `payload`, and why it was not read under `reason`, all marked by
`form = verbatim`:

```ini
[0]
form = verbatim
reason = Assertion error: Spawn packet is of spawn version 118, Shadow of Chernobyl's, whose layout is not supported
section = breakable_object
name = meshes\brkbl#0.ogf
script_game_id = 0
script_rp = 254
position = -4.1748385,12.251056,-9.147421
direction = -0.0009636891,-0.00012802181,-0.2570078
payload = AAD_______8hAHYABQAAAP__MQD__wAAAAABAAAA_____7r___8A__________9tZXNoZXNcYnJrYmwjMAAAAACAPw
```

`unpack` prints how many objects it wrote each way and groups the verbatim ones by reason; its `--report` result carries
the same as `typedObjects` and `verbatimObjects`. `pack` writes a verbatim object from its `payload` as it stands, and
ignores `reason`. Lists the game ships number their objects from zero; an object listed under another number keeps it in
a `chunk_id` key.

`pack`, `re-unpack` and `verify --unpacked` tell the two kinds of folder apart by what they hold: `header.ltx` for a
root spawn, `level_spawns.ltx` for a level's list. A folder holding neither, or both, is refused.

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
