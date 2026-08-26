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

## Command reference

{{#include reference/ltx.md:commands}}
