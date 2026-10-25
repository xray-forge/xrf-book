# Game materials CLI

Game material commands inspect `gamemtl.xr`, the library of physical materials that collision faces, bones and creatures
are made of, and of the pairs that decide the sounds, particles and marks where two materials meet.

## Examples

Run from a working directory containing the game's `gamemtl.xr`:

```powershell
xrf-cli gamemtl info --path ./gamemtl.xr
```

Example output — inspect the Call of Pripyat library:

```text
Read game material library ./gamemtl.xr
Game material library information:
Version: 1
Game: Call of Pripyat
Materials count: 63, dynamic: 29
Pairs count: 1421, inheriting: 984
```

Exit code: `0`.

Every game wrote version 1, so the game is told apart as the engine does: a library with no density chunk is taken for
Shadow of Chernobyl, one without the multiplayer shoot factor for Clear Sky. The game changes how the engine reads
ammunition, artefacts, bone protection and stalker fire. A library of another version is refused, as the engine refuses
it.

Dynamic materials are the ones moving objects and their bones are made of. Inheriting pairs name a parent pair that the
SDK editor takes their unset sounds, particles and marks from; the library stores the inherited lists, so the game reads
them as written.

To check the library against the rest of the game data, run the `materials` check of
[gamedata verification](gamedata.md).

{{#include reference/gamemtl.md:commands}}
