# OMF CLI

OMF commands inspect animation banks, verify serialization, and filter, rename, or duplicate motions. Use them when
adapting an imported bank to the motion names referenced by a weapon or HUD config.

Examples run from a working directory containing the named OMF files and JSON map. Filtering and renaming require an
explicit destination; duplication rewrites the source unless `--dest` is supplied.

## How motions are stored

An OMF holds parallel lists of motion definitions and keyframe payloads, paired by position. Definitions contain names
and playback parameters. The engine looks up a definition name, then reads its corresponding payload.

The editing commands maintain this pairing and update motion indices when necessary. LTX `anm_*` values refer to the
definition names.

## `omf info`

```powershell
xrf-cli omf info --path ./meshes/example.omf --verbose
```

Example output — inspect a motion bank:

```text
Read omf file ./gamedata/meshes/omf/wpn_mp5_hud_animation.omf
Omf file information
Version: 4
Motions: 3 mp5_shoot,idle,mp5_reload
Bones total: 7
Parts: default
Part 'default' bones: wpn_body,body,trigger,zatvor,lock,wpn_silencer,magazin
```

Exit code: `0`.

Inspection is read-only. It reports the version, motions, bone count, animation parts, and bones assigned to each part.
Verbose output adds per-motion keyframe counts, flags, speed, power, accrue, and falloff.

Keyframes and playback speed determine duration. Flag bit `0b10` means play once and stop; without that bit, a motion
loops. Vanilla idle motions commonly use `0b00`, while draw, shoot, and bore motions use `0b10`.

If a motion is missing, check the bank's names before changing the model or config that references it.

## `omf repack`

Select the mode by the input and output arguments:

| Input                | Operation                                                              |
| -------------------- | ---------------------------------------------------------------------- |
| File with `--dest`   | Read and write a re-serialized copy.                                   |
| File with `--verify` | Compare re-serialized bytes with the source in memory.                 |
| Directory            | Recursively verify `.omf` files without writing; `--dest` is rejected. |

```powershell
xrf-cli omf repack --path ./meshes/example.omf --dest ./meshes/example.repacked.omf
xrf-cli omf repack --path ./meshes/example.omf --verify
xrf-cli omf repack --path ./meshes
```

Example output — verify byte-identical serialization:

```text
Byte identical: ./gamedata/meshes/omf/wpn_mp5_hud_animation.omf
```

Exit code: `0`. Verbose mode makes the successful comparison visible.

The writer preserves chunk order and nested motion chunk ids. Verification requires byte-identical output, making it
useful after changing OMF parsing or serialization. A mismatch or processing error produces a non-zero exit code;
directory mode reports both counts.

Without verbose logging, a passing single-file verification is silent and a directory run prints failures and its
summary. Add `--verbose` to list files that matched.

## `omf filter-motions`

Extract the motions needed from a shared bank:

```powershell
xrf-cli omf filter-motions --path ./shared_bank.omf --dest ./wpn_ak74_hud_animation.omf `
  --keep-prefix ak_74_
xrf-cli omf filter-motions --path ./bank.omf --dest ./trimmed.omf --keep idle `
  --keep-prefix ak_74_ pist_
```

At least one exact `--keep` name or literal `--keep-prefix` is required. A motion survives if it matches any selector;
`ak_74_` and `ak74_` select different names. Matching nothing fails before writing.

Surviving definitions and payloads retain their pairing, with motion indices renumbered to their new positions. Use a
separate destination to retain the shared bank for other weapons. Add `--dry-run` to inspect the selection without
writing, or `--verbose` to list the resulting motions.

## `omf rename-motions`

Create `ak74.json` as a flat map from current names to replacement names:

```json
{
  "ak_74_draw": "ak74_draw",
  "ak_74_idle_move": "ak74_idle_moving",
  "ak_74_grenade_off": "ak74_switch_off"
}
```

Apply it to the bank:

```powershell
xrf-cli omf rename-motions --path ./trimmed.omf --dest ./renamed.omf --map ./ak74.json
```

Unmapped motions keep their names. Add `--strict` to require a map entry for every motion; the error identifies missing
entries. A map matching nothing or producing duplicate motion names is refused before writing.

Renaming updates definition and payload names together. Use `--dry-run` to preview the change and `--verbose` to list
the result, then update configs that refer to the old names.

## `omf duplicate-motion`

Copy an existing motion under a new name:

```powershell
xrf-cli omf duplicate-motion --path ./wpn_hand_pm_hud_animation.omf --from pm_idle `
  --to pm_idle_bore --play-once
```

This example edits the bank in place. Supply `--dest` to keep the original. Both definition and keyframe payload are
copied into a new paired slot, increasing the file by one motion's payload. An unknown source name or an existing
destination name is refused.

`--play-once` sets the stop-at-end flag on the copy while preserving other flags.

### Why `--play-once` exists

The weapon bore state returns to idle through its animation-end callback. Pointing `anm_bore` at a looping motion can
leave the weapon in that state until another action forces a transition.

When an imported bank has no bore motion, a play-once copy of its idle can hold the pose and then deliver the end
callback. Confirm with `omf info --verbose` that the copy exists and its `0b10` flag bit is set, then check the state
transition in game.

## Failure notes

A parse failure stops editing before writing. For a truncated chunk, re-extract the file from its original packed
archive and retry inspection; repacking cannot reconstruct missing bytes.

The writer rejects data that the selected OMF version cannot represent. Motion marks require version 4, so version 3
data carrying marks fails. A mismatch between definition and payload counts also fails.

After editing, inspect the bank with `omf info`, confirm its model's references with [`ogf info`](ogf.md#ogf-info), and
verify the assembled gamedata. A valid bank alone does not prove that every caller uses its new names.

## Command reference

{{#include reference/omf.md:commands}}
