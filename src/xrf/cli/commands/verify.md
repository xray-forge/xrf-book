# Verify

`verify` validates project setup and generated data.

```powershell
npm run cli -- verify <command>
```

`npm run verify` runs `verify project`.

## Commands

| Command                     | Checks                              |
| --------------------------- | ----------------------------------- |
| `verify project`            | Project setup and links.            |
| `verify gamedata`           | Assembled `target/gamedata`.        |
| `verify externs`            | Tracked extern manifest.            |
| `verify ltx`                | LTX structure and `$scheme` values. |
| `verify particles-packed`   | Packed `particles.xr`.              |
| `verify particles-unpacked` | Unpacked particle files.            |
| `verify translations`       | Project translation dictionaries.   |

## Options

- `verify gamedata -c, --checks <checks...>`: run only the listed checks instead of all of them.
- `verify gamedata -r, --report <report>`: write the structured verification report as JSON.
- `verify gamedata -v, --verbose`: print verbose external-tool logs.
- `verify gamedata -s, --strict`: fully validate expensive asset payloads, including complete sound decoding.
- `verify ltx -v, --verbose`: print verbose external-tool logs.
- Particle verification commands support `-v, --verbose`.
- `verify translations -l, --language <locale>`: check one locale instead of all of them.
- `verify translations -s, --strict`: fail on missing entries instead of only listing them.
- `verify translations -v, --verbose`: print verbose external-tool logs.

### Selecting checks

A full run validates everything and is slow. When iterating on one kind of asset, narrow it:

```powershell
npm run cli -- verify gamedata --checks meshes weapons animations
```

Available checks: `animations`, `levels`, `ltx`, `meshes`, `particles`, `particles-usage`, `scripts`, `shaders`,
`sounds`, `spawns`, `textures`, `weapons`, `weathers`. Unknown names are rejected before the tool runs.

### Structured report

`--report` writes the findings as JSON instead of leaving them only in the log. Because the format is stable, a report
from a known-good build can be kept as a baseline and later runs compared against it, which is more reliable than
reading console output when a change is expected to alter some findings but not others.

```powershell
npm run cli -- verify gamedata --checks weapons --report target/verify-weapons.json
```

## Examples

```powershell
npm run cli -- verify externs
npm run cli -- verify gamedata --verbose
npm run cli -- verify translations --language ukr --strict
```

Build gamedata before `verify gamedata`. `verify externs` does not write files. Regenerate a stale manifest with
`npm run cli -- build --include externs`.

## Failure notes

`verify project` reports setup problems without failing. Other checks fail on invalid data or tool errors.
`verify translations` reads `src/engine/translations`, not built gamedata, and only fails with `--strict`.
