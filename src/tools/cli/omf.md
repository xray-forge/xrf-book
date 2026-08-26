# OMF CLI

OMF commands inspect, re-serialize, and edit the motion set of X-Ray motion files.

## Commands

| Command                | Purpose                                                         | Writes files       |
| ---------------------- | --------------------------------------------------------------- | ------------------ |
| `omf info`             | Print version, motions, bones, and animation parts.             | No                 |
| `omf repack`           | Read a motion file and write it back, or verify it round-trips. | Only with `--dest` |
| `omf filter-motions`   | Keep only selected motions, dropping the rest.                  | Yes                |
| `omf rename-motions`   | Rename motions using a name map.                                | Yes                |
| `omf duplicate-motion` | Copy a motion under a new name, optionally clearing its loop.   | Yes                |

## How motions are stored

Both editing commands rely on the same structure, which is worth knowing before using them.

A motion file holds two parallel lists: motion **definitions**, which carry the name and playback parameters, and motion
**payloads**, which carry the keyframes. They are paired by position. The engine looks a motion up by its definition
name and then reads the payload stored at the same position, so the two lists must always be filtered and renamed
together. Both commands maintain that pairing; the names an LTX `anm_*` key refers to are the definition names.

## `omf info`

```powershell
xrf-cli omf info --path ./meshes/example.omf
```

Options:

- `-p, --path <path>`: path to an `.omf` file. Required.

### Output

The command reads the motion file and prints:

- OMF version;
- motion count and motion names;
- total bone count;
- animation part names;
- bones assigned to each animation part.

With `-v`, each motion is listed individually with its keyframe count, flags, speed, power, accrue, and falloff.
Keyframes and speed together give the effective duration. The flags matter more than they look: bit `0b10` means the
motion plays once and stops, and a motion without it loops forever. Vanilla idles read `0b00` while draw, shoot and bore
motions read `0b10`.

### When to use it

Use `omf info` when checking whether a motion file is readable, whether expected motions are present, or how motion
parts map to skeleton bones.

The command is read-only and prints the parsed structure; it does not merge, split, or repair motion files. If an
expected animation is absent, check the source OMF first and then inspect the model or config that references the motion
name.

## `omf repack`

```powershell
xrf-cli omf repack --path ./meshes/example.omf --dest ./meshes/example.repacked.omf
xrf-cli omf repack --path ./meshes/example.omf --verify
xrf-cli omf repack --path ./meshes
```

Options:

- `-p, --path <path>`: path to an `.omf` file, or to a directory containing `.omf` files. Required.
- `-d, --dest <dest>`: path to the resulting `.omf` file. Required when writing a single file. Rejected when `--path` is
  a directory.
- `--verify`: compare the re-serialized bytes against the source instead of writing output.

The command has three modes:

- **Write.** A file path plus `--dest` reads the source and writes a re-serialized copy.
- **Verify one file.** A file path plus `--verify` re-serializes in memory and compares against the source bytes.
  Nothing is written.
- **Verify a directory.** A directory path walks it recursively, verifies every `.omf` beneath it, and prints a summary.
  Nothing is written, and `--dest` is rejected.

Unmodified files round-trip byte for byte. The writer preserves the original chunk order and reproduces the nested
motion chunk ids, so a repacked file that differs from its source indicates a real change, not serialization noise.

### When to use it

Use directory verification as a regression check after changing OMF parsing or serialization, and before relying on
generated motion banks. Because it fails on any mismatch, it works as a build or pre-commit gate.

Use single-file write mode when you need a normalized copy of a motion file.

## `omf filter-motions`

Shared animation banks often carry motions for many weapons at once. This command extracts the subset you need.

```powershell
xrf-cli omf filter-motions --path ./shared_bank.omf --dest ./wpn_ak74_hud_animation.omf --keep-prefix ak_74_
xrf-cli omf filter-motions --path ./bank.omf --dest ./trimmed.omf --keep idle --keep-prefix ak_74_ pist_
```

Options:

- `-p, --path <path>`: source `.omf` file. Required.
- `-d, --dest <dest>`: resulting `.omf` file. Required.
- `-k, --keep <name>...`: exact motion names to keep.
- `--keep-prefix <prefix>...`: keep motions whose name starts with the prefix.
- `--dry-run`: report the outcome without writing anything.

At least one of `--keep` or `--keep-prefix` must be given, and a motion is kept if it matches any of them. Prefixes are
matched literally, so `ak_74_` and `ak74_` select different motions.

Surviving definitions are renumbered so their internal motion index stays consistent with their new position.

`--dest` is required rather than defaulting to an in-place rewrite. Trimming a shared bank in place would destroy it for
every other weapon that sources from it, and the input is often a read-only reference dump.

## `omf rename-motions`

```powershell
xrf-cli omf rename-motions --path ./trimmed.omf --dest ./renamed.omf --map ./ak74.json
xrf-cli omf rename-motions --path ./trimmed.omf --dest ./renamed.omf --map ./ak74.json --strict
```

Options:

- `-p, --path <path>`: source `.omf` file. Required.
- `-d, --dest <dest>`: resulting `.omf` file. Required.
- `-m, --map <map>`: JSON object mapping existing motion names to new ones. Required.
- `--strict`: require every motion in the file to appear in the map.
- `--dry-run`: report the outcome without writing anything.

The map is a flat JSON object:

```json
{
  "ak_74_draw": "ak74_draw",
  "ak_74_idle_move": "ak74_idle_moving",
  "ak_74_grenade_off": "ak74_switch_off"
}
```

Motions absent from the map keep their current name. Pass `--strict` when a file is meant to be fully normalized; it
fails and lists the motions that have no entry, which is the difference between a deliberate partial rename and an
incomplete map.

Renaming updates the definition name and the payload name together, so the new name is what the engine resolves.

## `omf duplicate-motion`

Copies one motion under a second name, so a bank can provide an animation it does not ship.

```powershell
xrf-cli omf duplicate-motion --path ./wpn_hand_pm_hud_animation.omf --from pm_idle --to pm_idle_bore --play-once
```

Options:

- `-p, --path <path>`: path to an `.omf` file. Required.
- `-d, --dest <dest>`: path to the resulting file. Defaults to rewriting the source file in place.
- `--from <name>`: motion to copy, matched exactly. Required.
- `--to <name>`: name to give the copy. Required.
- `--play-once`: clear looping on the copy so it plays once and ends.

Both the definition and the keyframe payload are duplicated rather than aliased, and the copy is pointed at its own new
payload slot. That is deliberate: motion definitions and payloads are ordinal pairs, and a definition without a payload
would break every later filter or rename. The file grows by one animation's worth of keyframes.

An unknown `--from`, or a `--to` that already exists, is refused rather than producing an unreachable motion.

### Why `--play-once` exists

Some engine states are left **only** from the animation end callback. Bore is the one that bites: `OnAnimationEnd`
switches it back to idle, and nothing else does. Point `anm_bore` at a looping motion and the weapon sits in the bore
state indefinitely, appearing frozen until some other action forces a state change.

That is the situation when an imported animation pack ships no bore at all. Copying the weapon's idle and clearing its
loop yields a motion that holds the idle pose and then ends, which is enough for the engine to leave the state. Confirm
the result with `omf info -v`: the copy should read `0b10`.

## Shared options

All commands accept `-s, --silent` to disable logging and `-v, --verbose` to enable verbose logging. Under
`omf filter-motions` and `omf rename-motions`, `--verbose` prints the resulting motion list.

Under `omf repack`, `--verbose` additionally reports every file that verified as byte identical. Without it, verifying a
directory prints only mismatches, read failures, and a closing summary, and verifying a single file that passes prints
nothing at all. Scripts can therefore treat single-file verification as silent on success and rely on the exit status.

## Failure notes

`omf repack` exits with a non-zero status when any file mismatches or cannot be processed. In directory mode it also
reports the count of mismatched and errored files. A successful run exits zero.

A file that cannot be read fails before anything is written. The most common cause is a truncated source: a chunk header
declares more bytes than the file actually contains. Such files are damaged at the source and need to be re-extracted,
usually from the original packed archive with `archive unpack` rather than from an unpacked dump.

Writing rejects data it cannot represent faithfully rather than emitting a corrupt file. Motion marks exist only in
version 4, so writing a version 3 file that carries marks fails, as does writing a file whose motion count and motion
definition count disagree.

The editing commands refuse work that would produce a file the engine cannot use, before writing anything:

- `omf filter-motions` fails when no motion matches, since an empty motion file is not useful and the usual cause is a
  mistyped prefix.
- `omf rename-motions` fails when the map matches nothing, and when a rename would give two motions the same name,
  because lookups are by name and a duplicate makes one of them unreachable.

## Command reference

{{#include reference/omf.md:commands}}
