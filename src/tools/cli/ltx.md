# LTX CLI

LTX commands inspect, format, and verify `.ltx` and `.ini` config files. Use them for standalone config projects or when
you need the lower-level tool behind the engine repository's `format ltx` and `verify ltx` commands.

## Formatting

`ltx format` formats selected loose config files. `--check` writes nothing and exits 3 when any selected file needs
formatting, including when the input is a single file. From a project containing `gamedata/configs`:

```powershell
xrf-cli ltx format --path ./gamedata/configs
xrf-cli ltx format --path ./gamedata/configs --check
```

When the input resolves through an installation, archived configs are listed as declined because they cannot be
rewritten in place. Review that list before treating a formatting run as coverage of the whole config set.

## Verifying

`ltx verify` checks an LTX project folder, including schemes and case-sensitive include paths. Supply a directory;
include, inheritance, section-field, and scheme errors fail verification.

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

Use the resolved values and their source locations to distinguish a wrong declaration from a later override. If several
entry points declare the section, select one with `--entry system.ltx`. Both commands accept `--dltx` for patched
installations.

## The DLTX patch dialect

Anomaly and its Monolith-based descendants let an addon patch a config without editing it, by dropping a
`mod_<base>_*.ltx` beside it. `--dltx` reads configs under those rules; without it, a patch file is refused and the
error names the flag.

```powershell
xrf-cli ltx verify --path "C:/games/anomaly" --dltx
xrf-cli gamedata verify "C:/games/anomaly" --dltx
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
