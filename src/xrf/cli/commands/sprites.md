# Sprites

`sprites` wraps sprite tooling from `cli/bin/tools/xrf-cli`. It packs and unpacks the equipment sprite and the UI
texture description sprites using project paths from `cli/config.json`.

```powershell
npm run cli -- sprites <command>
```

## Commands

| Command                      | Reads                                                                                 | Writes                                                 |
| ---------------------------- | ------------------------------------------------------------------------------------- | ------------------------------------------------------ |
| `sprites unpack-equipment`   | `src/resources/textures/ui/ui_icon_equipment.dds` and `src/engine/configs/system.ltx` | `src/resources/textures_unpacked/ui/ui_icon_equipment` |
| `sprites pack-equipment`     | unpacked equipment icons and `system.ltx`                                             | `src/resources/textures/ui/ui_icon_equipment.dds`      |
| `sprites unpack-description` | UI texture descriptions and packed textures                                           | `src/resources/textures_unpacked`                      |
| `sprites pack-description`   | UI texture descriptions and unpacked textures                                         | `src/resources/textures`                               |

## Options

All sprite commands support:

- `-v, --verbose`: print verbose logs.
- `-s, --strict`: enable strict mode.

Description commands also support:

- `-d, --description <name>`: process one file under `src/engine/forms/textures_descr`.

## Examples

```powershell
npm run cli -- sprites unpack-equipment
npm run cli -- sprites pack-equipment --strict
npm run cli -- sprites unpack-description --description ui_actor.xml
npm run cli -- sprites pack-description --description ui_actor.xml
```

## Workflow

Unpack before editing a sprite's icons or checking generated sprite coordinates. Pack after editing the unpacked files
or texture descriptions. Use `--description` when working on a single UI texture description file instead of the whole
description set.

Equipment commands are tied to the equipment sprite. Description commands are tied to XML texture description files
under `src/engine/forms/textures_descr`.

## Failure notes

Equipment commands depend on valid `system.ltx` icon coordinates. Description commands depend on XML description names
and matching source textures.
