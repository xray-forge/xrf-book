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
