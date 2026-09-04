# LTX CLI

LTX commands format and verify `.ltx` and `.ini` config files. Use them for standalone config projects or when you need
the lower-level tool behind the engine repository's `format ltx` and `verify ltx` commands.

## Formatting

`ltx format` formats one file or every LTX file under a folder. `--check` reports what is unformatted instead of
rewriting it, and answers 3 when anything is; single-file mode formats the file.

```powershell
xrf-cli ltx format --path ./gamedata/configs
xrf-cli ltx format --path ./gamedata/configs --check
```

## Verifying

`ltx verify` verifies an LTX project folder, including its schemes and case-sensitive include paths. It expects a
directory rather than a single file, and fails when includes, inheritance, section fields, or scheme validation produce
errors.

```powershell
xrf-cli ltx verify --path ./gamedata/configs
```

Only sections that declare `$schema` are checked against a scheme. Other sections, including array-style sections, are
valid without one. A scheme can use `$strict = true` when its own section shape is fully known.

Scheme definitions are documented in [Script config schemes](../../script_engine/configs_scheme.md).

## The DLTX patch dialect

Anomaly and its Monolith-based descendants let an addon patch a config without editing it, by dropping a
`mod_<base>_*.ltx` beside it. `--dltx` reads configs under those rules; without it, a patch file is refused and the
error names the flag.

```powershell
xrf-cli ltx verify --path "C:/games/anomaly" --dltx
xrf-cli gamedata verify "C:/games/anomaly" --dltx
```

DLTX is not vanilla LTX with patches applied on top. It changes how base data resolves even when no patch file exists:

| Behaviour         | Standard LTX                  | `--dltx`                                                    |
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
