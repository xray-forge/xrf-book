# Spawn

`spawn` contains ALife spawn file utilities. The engine CLI currently exposes the unpack workflow.

```powershell
npm run cli -- spawn unpack
```

## Defaults

| Field       | Default                          |
| ----------- | -------------------------------- |
| Source      | `src/resources/spawns/all.spawn` |
| Destination | `target/all_spawn`               |

The command delegates to `cli/bin/tools/xrf-cli spawn unpack`.

## Options

- `-p, --path <path>`: source spawn file path.
- `-d, --dest <dest>`: output directory.
- `-v, --verbose`: print verbose logs.
- `-f, --force`: remove an existing unpacked destination before writing.

## Examples

```powershell
npm run cli -- spawn unpack
npm run cli -- spawn unpack --force
npm run cli -- spawn unpack --path src/resources/spawns/all.spawn --dest target/all_spawn
```

## Output

The output directory contains the unpacked spawn representation produced by the XRF tools CLI. The engine wrapper is
intended for inspection and verification workflows in this repository; it does not expose pack or repack commands.

Keep generated unpack output under `target/` unless you are intentionally preparing source data for another tool.

## Failure notes

The source spawn file must exist. Use the Tools CLI spawn commands when you need lower-level spawn info, pack, repack,
or verification operations.
