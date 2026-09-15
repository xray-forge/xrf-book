# Translation CLI

Translation commands import X-Ray XML string tables, maintain XRF JSON translation sources, and compile them back to
gamedata. Start with [Parse](#parse) for an existing game or mod; use [Format](#format), [Verify](#verify), and
[Build](#build) for an existing JSON project.

Examples run from a working directory containing a `translations` source folder. Import examples additionally assume a
`stalker-anomaly` installation beside it.

## Initialize

Add missing language keys as `null` placeholders in the JSON sources:

```powershell
xrf-cli translation initialize --path ./translations
```

This updates files in place. Running it again on the same initialized sources makes no further change. A `null` marks a
missing translation; it does not supply fallback text.

## Format

Normalize source layout, then use check mode in automation:

```powershell
xrf-cli translation format --path ./translations
xrf-cli translation format --path ./translations --check
```

Example output — check unformatted JSON sources:

```text
Checking 2 translation source(s)
Not formatted: ./translations\st_items.json
Not formatted: ./translations\st_ui.json
```

Stderr:

```text
Format issues with 2/2 translation source(s) in 2 ms
Check failed: 2 finding(s)
```

Exit code: `3`.

The formatter sorts ids and language keys naturally, uses two-space indentation, and adds a trailing newline. Natural
order places `st_thanks2` before `st_thanks10`, and `ammo-5.45x39-ap` before `ammo-11.43x23-fmj`. Check mode writes
nothing and exits 3 when selected files need formatting.

Repeat `--path` for multiple inputs. Directories select JSON files recursively; an explicitly named file is accepted
regardless of its extension.

By default, each file uses its dominant LF or CRLF line ending; a tie or a file without line breaks selects LF. Mixed
line endings are normalized to that choice. Check mode ignores line-ending differences unless `--line-endings lf` or
`--line-endings crlf` explicitly requires one convention.

### What it does not touch

Formatting preserves JSON values and their representation as strings or arrays. A one-element `["text"]` stays an array.
The build joins array elements with the literal `\n` sequence used by string values, so formatting does not choose
between these authoring forms.

It does not add `null` placeholders; use `initialize` or `parse` for that. Files already matching the selected format
are not rewritten, preserving their timestamps.

### When it refuses

Selecting no sources or encountering an unparseable source exits 1. A parse failure stops the run at that file; files
formatted earlier remain changed. Each replacement is staged as a whole file.

## Build

Compile JSON sources into one XML string table per source and selected language, using that language's code page:

```powershell
xrf-cli translation build --path ./translations --output ./gamedata/configs/text --language ukr
```

Example output — build Ukrainian string tables:

```text
Building translations in ./translations (ContainingInstallation), language - ukr, sorted - true
Building 2 translation source(s)
Built translation files in 2 ms
```

Exit code: `0`.

A missing translation compiles to its id. The report summarizes tables written and ids compiled per language.

Build and verify accept a single source file or roots read through the virtual file system. Layered roots resolve
winning files by priority, including files from mounted archives. The build output is a plain directory and must be
outside every source root.

## Verify

Check completeness for a language:

```powershell
xrf-cli translation verify --path ./translations --language ukr --strict `
  --report ./translation-report.json
```

Example output — find missing Ukrainian text:

```text
Verifying translations in ./translations (ContainingInstallation), language - ukr
Verifying 2 translation source(s)
Verified translation files in 0 ms, 4 checked, 2 missing
```

Stderr:

```text
Translation key missing: st_medkit_name ukr in st_items.json
Translation key missing: st_ui_quit ukr in st_ui.json
Check failed: 2 finding(s)
```

Exit code: `3`. Both the absent key and the explicit null are reported as missing.

Both an absent language key and an explicit `null` count as missing. Without `--strict`, missing translations are
reported while a completed check succeeds. With `--strict`, those gaps produce exit 3.

The report contains a finding per missing id and a `languages` array with summary rows per file and language. Use the
summary rows to review large imports, then inspect findings for the files being translated. An unreadable source is an
execution failure; malformed source content can fail verification independently of `--strict`.

## Parse

Import XML tables once per language into a shared JSON output directory:

```powershell
xrf-cli translation parse --path ./stalker-anomaly --language eng --output ./translations
xrf-cli translation parse --path ./stalker-anomaly --language ukr --output ./translations
```

XML tables do not declare their language. `--language` labels the imported text, so select the language that the input
actually contains. Installations with tables in `db/configs` archives are read through the same virtual file system as
loose trees.

### What it writes

Each table becomes a JSON source with its subdirectory path preserved. Imports into the same output merge languages. Ids
and language keys use the same canonical order as [Format](#format), independently of import order. A record missing one
of the languages represented in its file receives an explicit `null`.

Existing text that differs from the import is preserved and counted as a conflict. Add `--overwrite` to replace it.
Reimporting unchanged tables into an unchanged output is idempotent.

### Finding the tables

`--path` names the input root. The importer looks under `configs/text` when present, then selects the directory named
for the requested language. Use `--prefix` for a different layout. A selected scope that still contains another
language's directory is refused to prevent labeling its strings with the wrong language.

### Before writing anything

Add `--dry-run` to inspect the proposed import without writing. Use `--file` to select one table. Unreadable tables are
reported; `--strict` makes those findings fail the run.

## Notes

In the engine repository, `npm run cli -- verify translations` wraps verification, and the `translations` build target
wraps compilation. Call `xrf-cli` directly for formatting, initialization, and imports.

## Command reference

{{#include reference/translation.md:commands}}
