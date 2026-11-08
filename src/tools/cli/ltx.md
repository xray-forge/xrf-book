# LTX CLI

LTX commands list, search, inspect, format, and verify `.ltx` and `.ini` config files. Use them for standalone config
projects or when you need the lower-level tool behind the engine repository's `format ltx` and `verify ltx` commands.

## Formatting

`ltx format` formats selected loose config files. `--check` writes nothing and exits 3 when any selected file needs
formatting, including when the input is a single file. From a project containing `gamedata/configs`:

```powershell
xrf-cli ltx format --path ./gamedata/configs
xrf-cli ltx format --path ./gamedata/configs --check
```

Example output — check an unformatted LTX file:

```text
Checking 1 ltx file(s) from 1 provided path(s)
Checking 1 file(s)
Not formatted: ./ltx-unformatted/spacing.ltx
```

Stderr:

```text
Format issues with 1/1 files in 2 ms
Check failed: 1 finding(s)
```

Exit code: `3`.

When the input resolves through an installation, archived configs are listed as declined because they cannot be
rewritten in place. Review that list before treating a formatting run as coverage of the whole config set.

## Verifying

`ltx verify` checks an LTX project folder, including schemes and case-sensitive include paths. Supply a directory;
include, inheritance, section-field, and scheme errors fail verification. So does an entry point that does not resolve,
and, under `--dltx`, a statement the game loads and then silently ignores, such as an override of a section nothing
declares. `--report` writes the verify commands' shared shape: the counts beside one `ltx` check, whose findings name
the config to open and lead with the section and field.

```powershell
xrf-cli ltx verify --path ./gamedata/configs
```

Only sections that declare `$scheme` are checked against a scheme. Other sections, including array-style sections, are
valid without one. A scheme can use `$strict = true` when its own section shape is fully known.

Scheme definitions are documented in [Script config schemes](../../script_engine/configs_scheme.md).

## Inspecting configs

Use `ltx list` to list config files and their includers. Use `ltx inspect` to see a resolved section's values and where
each value was written:

```powershell
xrf-cli ltx list --path ./gamedata/configs
xrf-cli ltx inspect wpn_ak74 --path ./gamedata/configs
```

Example output — inspect a DLTX override:

```text
Inspect path: ./gamedata-dltx/configs
[wpn_ak74] resolved from system.ltx (dltx)
  declared in items\w_ak74.ltx
  inherits wpn_base
  $scheme    = $wpn_patched          set by w_ak74.ltx (depth 1)
  ammo_class = ammo_a,ammo_b,ammo_c  set by mod_system_aaa.ltx ('>', depth -200)
  cost       = 9000                  set by mod_system_xxx.ltx (depth -400)
  patched_by = mod_system_xxx.ltx    set by mod_system_xxx.ltx (depth -400)
4 field(s), 0 diagnostic(s)
```

Exit code: `0`. The later patch supplies cost = 9000; the output names the file that supplied each value.

Use the resolved values and their source locations to distinguish a wrong declaration from a later override. If several
entry points declare the section, select one with `--entry system.ltx`. Both commands accept `--dltx` for patched
installations.

## Finding where a section or field is written

Use `ltx find` to list every place a section or field is written, including configs inside archives and DLTX patch
files. Give it `--section`, `--field`, `--value`, or any combination:

```powershell
xrf-cli ltx find --path ./gamedata/configs --section wpn_ak74
xrf-cli ltx find --path ./gamedata/configs --field ammo_class --value ammo_c
```

Example output — find a section that patch files override:

```text
Find path: ./gamedata-dltx/configs
items\w_ak74.ltx:1 [wpn_ak74]:wpn_base - declaration
mod_system_aaa.ltx:3 ![wpn_ak74] - patch
mod_system_xxx.ltx:3 ![wpn_ak74] - patch
Found 3 statement(s) in 3 config(s): 1 declaration(s), 2 patch(es)
Resolved values: ltx inspect --path ./gamedata-dltx/configs --dltx wpn_ak74
```

Exit code: `0`, also when nothing matches.

Each line names the config and line to open, the statement as written with its operation prefix, and whether it declares
or patches. A field written under a `![section]` header counts as a patch. `[archived]` marks a config read from an
archive, which you cannot edit in place.

Section and field names must match exactly, including case, as the game compares them. `--value` matches any value that
contains the text, also case-sensitively. Pass names alone: `--section [wpn_ak74]` or `--field !rpm` is refused, and
`--field rpm` already finds `!rpm`, `>rpm` and `<rpm`. A section query alone lists headers; adding `--field` or
`--value` lists field lines instead.

`ltx find` shows what each file writes, not the value the game ends up with. Several files can write the same field;
`ltx inspect` names the one that wins.

## The DLTX patch dialect

Anomaly and its Monolith-based descendants let an addon patch a config without editing it, by dropping a
`mod_<base>_*.ltx` beside it. `--dltx` reads configs under those rules; without it, a patch file is refused and the
error names the flag.

```powershell
xrf-cli ltx verify --path "C:/games/anomaly" --dltx
xrf-cli gamedata verify --path "C:/games/anomaly" --dltx
```

DLTX is not vanilla LTX with patches applied on top. It changes how base data resolves even when no patch file exists:

| Behavior          | Standard LTX                  | `--dltx`                                                    |
| ----------------- | ----------------------------- | ----------------------------------------------------------- |
| Include priority  | Read order                    | By depth, so a root file beats a file it includes           |
| Inheritance       | Parent must be declared first | Resolved after the whole tree is read, forward refs allowed |
| Missing parent    | Refuses                       | Contributes nothing, and XRF warns where the game is silent |
| Duplicate section | Refuses                       | Refuses, unless marked an override with `![section]`        |

Patch operations, all Monolith-specific:

| Statement      | Effect                                                  |
| -------------- | ------------------------------------------------------- |
| `![section]`   | Override an existing section                            |
| `@[section]`   | Override it, creating it first when nothing declares it |
| `!![section]`  | Delete it, after everything else resolves               |
| `!key`         | Delete a field                                          |
| `>key = a, b`  | Append to a comma list                                  |
| `<key = a, b`  | Remove from a comma list                                |
| `[section]:!p` | Drop an inherited parent                                |

When several patch files touch the same field, the **alphabetically last one wins**, and a patch file always outranks
the base tree.

## Command reference

{{#include reference/ltx.md:commands}}
