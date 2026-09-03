# Archive CLI

Use the archive commands to pack a `gamedata` directory into X-Ray `.db` files, inspect an existing archive, or unpack
and verify it. Start with `archive pack` when you are building a database; use the read commands when you only need to
examine one.

## Pack an archive

Pack a `gamedata` tree with a name and destination of your choice:

```powershell
xrf-cli archive pack --path target\gamedata --dest target\db --name gamedata
```

The command compresses file types the engine normally compresses and stores the rest. It writes one volume as
`gamedata.db`; when the archive needs more than one volume, it writes `gamedata.db0`, `gamedata.db1`, and so on.

By default, a volume can be up to 1900 MB and receives a header that mounts its contents at `$fs_root$\gamedata\`. That
is the usual setting for a `gamedata` archive. To use a different mount point, supply the complete header yourself:

```powershell
xrf-cli archive pack --path target\levels --dest target\db --name levels `
  --header 'auto_load=true' --header 'entry_point=$fs_root$\levels\'
```

Each `--header` value is `key=value`. Supplying any header entries replaces the default header, so include both
`auto_load` and `entry_point` when you need the standard behavior with a different entry point.

### Choose what to pack

Without selection options, the command packs the whole source directory. Use a configuration file when the selection is
shared or checked in; use command-line options for a one-off build. They cannot be combined.

An `.ltx` configuration uses the `xrCompress` dialect:

```ini
[options]
exclude_exts = *.txt,*.json

[include_folders]
configs = true
scripts = true

[include_files]
gamemtl.xr

[header]
auto_load = true
entry_point = $fs_root$\gamedata\
```

```powershell
xrf-cli archive pack --path target\gamedata --dest target\db --name gamedata `
  --config pack.ltx
```

In `[include_folders]` and `[exclude_folders]`, `true` applies to the directory and everything below it; `false` applies
only to the named directory. Use `.\` for the packed root. An `.ltx` or `.json` configuration may contain selection
rules and a header only. Source path, destination, volume name, and run options remain on the command line.

A JSON configuration for the same selection looks like this:

```json
{
  "excludeExtensions": ["*.txt", "*.json"],
  "includeFiles": ["gamemtl.xr"],
  "includeDirectories": [
    { "path": "configs", "isRecursive": true },
    { "path": "scripts", "isRecursive": true }
  ],
  "header": [
    { "key": "auto_load", "value": "true" },
    { "key": "entry_point", "value": "$fs_root$\\gamedata\\" }
  ]
}
```

```powershell
xrf-cli archive pack --path target\gamedata --dest target\db --name gamedata `
  --config pack.json
```

For a direct selection, repeat the relevant option:

```powershell
xrf-cli archive pack --path target\gamedata --dest target\db --name configs `
  --include-directory configs --include-directory spawns `
  --include-file gamemtl.xr --exclude-extension '*.txt'
```

`--include-directory-shallow` includes a directory's files but not the files in its child directories.
`--exclude-directory-shallow` excludes the named directory only; its contents can still be packed. All paths are
relative to `--path`.

### Common packing options

- Use `--store` to store every file without compression.
- Use `--max-size <MB>` to choose a volume cap from 1 through 1900 MB. `--oversized-volumes` permits a larger cap only
  for an engine fork that supports it.
- Use `--xdb` to create `.xdb` volumes.
- Use `--no-skip-list` to retain editor and source leftovers that the normal engine-build skip list excludes.
- Use `--verbose` to see every selected, skipped, stored, compressed, and deduplicated file while packing.

### Performance compared with xrCompress

These are median results from interleaved runs of both tools on the same machine and source tree. `xrCompress -fast`
uses the compression mode that matches `archive pack`; the xrCompress default trades time for a smaller archive. Each
time/RAM value is wall-clock seconds and peak resident memory in megabytes.

| Input                                    | `archive pack` time / peak RAM (s / MB) | `xrCompress -fast` time / peak RAM (s / MB) |
| ---------------------------------------- | --------------------------------------- | ------------------------------------------- |
| 1,657 config files, 9.89 MB              | 0.22 s / 11 MB                          | 0.57 s / 99 MB                              |
| 4,206 Anomaly configs and scripts, 35 MB | 0.86 s / 13 MB                          | 0.97 s / 101 MB                             |
| 1,017 mesh files, 275 MB                 | 0.17 s / 28 MB                          | 1.07 s / 115 MB                             |
| Vanilla gamedata, 36,925 files, 4.69 GB  | 4.6 s / 180 MB                          | 16.8 s / 275 MB                             |

For inputs that contain compressible files, these are the resulting archive sizes:

| Input                              | `archive pack` | `xrCompress -fast` | `xrCompress` |
| ---------------------------------- | -------------- | ------------------ | ------------ |
| 1,657 config files, 9.89 MB        | 2.00 MB        | 2.49 MB            | 1.93 MB      |
| Anomaly configs and scripts, 35 MB | 8.57 MB        | 10.63 MB           | 8.26 MB      |

Archives packed from the same source by either tool unpack to byte-identical files.

### Replace an existing archive

Packing refuses to overwrite volumes with the same name. Add `--force` only when replacing that set is intended:

```powershell
xrf-cli archive pack --path target\gamedata --dest target\db --name gamedata --force
```

`--force` is destructive. If that run fails partway through, the previous set cannot be restored automatically. A
non-forced run removes any volumes it created when it fails, leaving an existing different-named set alone.

## Inspect or extract files

For `info`, `list`, `find`, `extract`, and `verify`, `--path` may name one volume or a directory. A volume reads only
that file; a directory reads all `.db` and `.xdb` volumes below it as one merged archive set.

```powershell
# Check the number of volumes, entries, and their sizes.
xrf-cli archive info --path .\db

# List file paths, or search their names without unpacking.
xrf-cli archive list --path .\db --files
xrf-cli archive find --path .\db --query wpn_ak74 --files

# Extract one logical file, or an entire logical directory.
xrf-cli archive extract --path .\db --file textures\wpn\wpn_ak74.dds --dest .\ak74.dds
xrf-cli archive extract --path .\db --directory configs --dest .\extracted-configs
```

`list --verbose` and `find --verbose` show a file's sizes and source volume. If identical files share one stored
payload, they also name the other paths that read those bytes.

## Unpack an archive

Unpack a complete volume set by giving its containing directory:

```powershell
xrf-cli archive unpack --path .\db --dest .\unpacked\gamedata
```

To unpack one volume by itself, pass the volume path instead. `--dry` opens the archive and prints its summary without
writing files. Use `-j` to control the worker count, for example `-j 8` or `-j 50%`.

```powershell
xrf-cli archive unpack --path .\db\configs.db --dest .\unpacked\configs --dry
```

Use a new or empty destination directory. Existing files can otherwise be replaced while the archive is unpacked.

### Unpacking speed and memory

These results use the same measurement method. The default run uses the available worker count; `-j 1` is the
single-worker comparison.

| Archive                                 | `archive unpack` time / peak RAM (s / MB) | `-j 1` time / peak RAM (s / MB) |
| --------------------------------------- | ----------------------------------------- | ------------------------------- |
| Vanilla configs, 1,657 files, 2.00 MB   | 0.20 s / 12 MB                            | 0.40 s / 10 MB                  |
| Vanilla gamedata, 36,925 files, 4.48 GB | 6.1 s / 28 MB                             | 12.3 s / 21 MB                  |

## Verify an archive

Verify every file after packing or copying an archive:

```powershell
xrf-cli archive verify --path .\db
```

The command reads every payload, checks decompression, and validates its CRC. It reports damaged files as failures; use
`--json` or `--report archive-verify.json` when another tool needs the result.

## Command reference

{{#include reference/archive.md:commands}}
