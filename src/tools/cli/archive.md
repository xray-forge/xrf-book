# Archive CLI

Archive commands work with X-Ray `.db` database archives.

## Packing

`archive pack` builds database archives from a folder, replacing the `xrCompress` tool of the original SDK.

```powershell
xrf-cli archive pack --path target\gamedata --dest target\db --name gamedata
```

A set that fits in one volume is written as `<name>.db`; a larger one splits into `<name>.db0`, `<name>.db1` and so on.
The engine mounts any file whose extension starts with `db` or `xdb`, so the index is a convenience rather than a
requirement. Files the engine expects compressed are compressed and the rest are stored, matching what the engine loads.
Identical files are stored once and referenced twice.

Give the archive a `[header]`. Without one the engine assumes a `.db` is an encrypted Shadow of Chernobyl archive and
decrypts it into nonsense; `--xdb` is the other way to say an archive is not that.

### Replacing an existing set

A destination that already holds volumes of the same name is refused, and `--force` is how you say to replace them.
Packing writes each volume at its final name, so a forced run that fails or is stopped partway leaves neither the
previous set nor a complete new one. A run that was not forced takes back the volumes it made, so a failure leaves the
destination as it found it. Volumes of a different set name in the same directory are never touched: packing `gamedata`
and `textures` into one folder is ordinary.

### Configuration file

Without `--ltx` the whole folder is packed. The file uses the same dialect `xrCompress` accepted:

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

`[include_folders]` and `[exclude_folders]` map a path to whether it applies recursively; `.\` names the packed root.
`[include_files]` lists names one per line. `[header]` is written into the archive verbatim, and its `entry_point` is
where the engine mounts the contents, so an archive of a `gamedata` tree needs it.

Anything named on the command line wins over the configuration file.

### Compared with xrCompress

Both tools packing the same tree with the same configuration, on one machine, as the median of interleaved runs. The
`-fast` setting is the one that matches what the packer does; the xrCompress default trades speed for a smaller archive.

| Input                                    | `archive pack` | `xrCompress -fast` |
| ---------------------------------------- | -------------- | ------------------ |
| 1,657 config files, 9.89 MB              | 0.22 s         | 0.59 s             |
| 4,206 Anomaly configs and scripts, 35 MB | 0.89 s         | 1.00 s             |
| 1,017 mesh files, 275 MB                 | 0.21 s         | 1.06 s             |
| Vanilla gamedata, 36,925 files, 4.69 GB  | 5.9 s          | 18.2 s             |

Archive size, where the input holds anything compressible:

| Input                              | `archive pack` | `xrCompress -fast` | `xrCompress` |
| ---------------------------------- | -------------- | ------------------ | ------------ |
| 1,657 config files, 9.89 MB        | 2.00 MB        | 2.49 MB            | 1.93 MB      |
| Anomaly configs and scripts, 35 MB | 8.57 MB        | 10.63 MB           | 8.26 MB      |

The archives are interchangeable: packed from one tree by either tool, they unpack to byte-identical files.

## Unpacking

`archive unpack` opens an archive project and exports the contained files to a folder. Relative destination paths are
resolved from the current working directory.

```powershell
xrf-cli archive unpack --path .\db\configs.db0 --dest .\unpacked\configs
xrf-cli archive unpack --path .\db\textures.db0 --dest .\unpacked\textures --parallel 8
xrf-cli archive unpack --path .\db\sounds.db0 --dry
```

`--dry` still reads the archive metadata but writes no files. Use it to confirm that a database can be opened before
spending time on a full unpack.

The source path must point to a readable X-Ray database archive. If the destination already contains files, choose a new
folder or clean it before running the command.

### Speed

The original SDK shipped no unpacker, so these are absolute figures rather than a comparison. One machine, median of
interleaved runs, warm file cache.

| Archive                                  | Files  | Default | `--parallel 1` |
| ---------------------------------------- | ------ | ------- | -------------- |
| Vanilla configs, 2.00 MB                 | 1,657  | 0.28 s  | 0.72 s         |
| Anomaly configs and scripts, 8.57 MB     | 4,206  | 0.71 s  | 1.86 s         |
| Vanilla meshes, 275 MB                   | 1,017  | 0.28 s  | 0.75 s         |
| Vanilla gamedata, 4.48 GB over 3 volumes | 36,925 | 8.1 s   | 24.6 s         |

Unpacking spreads across cores and packing does not, so `--parallel` is the flag that moves these figures.

## Reading without unpacking

`archive info`, `archive list`, and `archive find` describe a volume or a whole set without writing anything, and
`archive extract` pulls out one file or one directory.

```powershell
xrf-cli archive info --path .\db
xrf-cli archive list --path .\db\configs.db0 --files
xrf-cli archive find --path .\db --query wpn_ak74
xrf-cli archive extract --path .\db --file textures\wpn\wpn_ak74.dds --dest .\ak74.dds
```

A path naming one volume reads that volume alone; a path naming a directory reads every volume it holds as one set.

## Verifying

`archive verify` reads every payload back and checks its decompression and CRC, so a set that opens but cannot be read
is reported rather than discovered later by the engine.

```powershell
xrf-cli archive verify --path .\db --json
```

## Command reference

{{#include reference/archive.md:commands}}
