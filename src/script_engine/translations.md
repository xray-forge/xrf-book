# Translations

Translations provide string-table source for UI labels, dialogs, tasks, item names, achievements, subtitles, and other
text shown by the game.

XRF keeps the translation source in JSON files under `src/engine/translations`. The build converts those sources into
X-Ray string-table XML under `target/gamedata/configs/text`.

## Supported languages

The supported locale keys are defined in `cli/config.json`:

| Key   | Language  |
| ----- | --------- |
| `eng` | English   |
| `fra` | French    |
| `ger` | German    |
| `ita` | Italian   |
| `pol` | Polish    |
| `rus` | Russian   |
| `spa` | Spanish   |
| `ukr` | Ukrainian |

The default build locale is also configured in `cli/config.json`. It can be overridden with the build command
`--language` option.

## Source format

Each JSON file is a dictionary keyed by translation id. Each translation id contains one value per supported locale.
Values can be strings or string arrays:

```json
{
  "st_example_name": {
    "eng": "Example",
    "fra": "Example",
    "ger": "Example",
    "ita": "Example",
    "pol": "Example",
    "rus": "Example",
    "spa": "Example",
    "ukr": "Example"
  }
}
```

String arrays are used when the generated XML text needs explicit line breaks. The build joins them on the `\n` the
engine reads as a line break, so an array and the single line it joins to produce the same string table.

## Canonical formatting

Sources have one canonical shape, and every tool that writes one produces it: ids and locale keys sorted, two-space
indentation, a trailing newline. Keeping to it is what stops a hand-added record and an imported one from producing
unrelated diff noise in the same file.

Ordering is natural rather than alphabetical by byte, so `st_thanks2` sorts before `st_thanks10` and `ammo-5.45x39-ap`
before `ammo-11.43x23-fmj`.

A source added or edited by hand is normalized with [`xrf-cli translation format`](../tools/cli/translation.md#format):

```powershell
xrf-cli translation format --path src/engine/translations
xrf-cli translation format --path src/engine/translations --check
```

`--check` writes nothing and exits non-zero when a source is not normalized, which is the form a build step or a
pre-commit hook uses. Formatting never changes what a source means: values are left exactly as written, and no locale
key is added or removed.

Line endings are not part of the canonical shape. Each file keeps the convention it already has, which is
`.gitattributes`' business rather than the formatter's.

## Importing existing string tables

JSON is the only source format. Raw X-Ray XML — a downloaded mod, a gamedata tree, or an installed game — becomes a
source through [`xrf-cli translation parse`](../tools/cli/translation.md#parse), one language per run:

```powershell
xrf-cli translation parse --path <mod or installation> --language eng --output src/engine/translations
xrf-cli translation parse --path <mod or installation> --language ukr --output src/engine/translations
```

Running it once per language merges them into one file per table, gives every record an explicit `null` for the
languages it lacks, and leaves text already in the source alone. Multi-line text is split into the array form above.

## Build behavior

The `translations` build target calls the bundled `xrf-cli` translation builder:

```powershell
npm run cli build -- --include translations
```

The source path is `src/engine/translations`. The target path is:

```text
target/gamedata/configs/text
```

Do not edit generated XML under `target/`. Fix the JSON source instead.

## Checking translations

The local CLI lists missing or invalid entries through `verify translations`:

```powershell
npm run cli -- verify translations
npm run cli -- verify translations --language eng
npm run cli -- verify translations --strict
```

Without `--strict` the command only reports gaps; `--strict` turns them into a non-zero exit.

Filling a new file with the full set of locale keys is an [`xrf-cli`](../tools/cli/translation.md) task:

```powershell
xrf-cli translation initialize --path src/engine/translations
```

## References from game data

Translation ids are referenced across multiple source types:

- UI form text nodes;
- dialog XML `text` nodes;
- task configs and task functors;
- item, weapon, outfit, and upgrade configs;
- script callbacks that return text ids.

When changing an id, search the repository before renaming it. Task fields and dialog conditions may be condlists or
script callbacks rather than plain string ids.

## Guidelines

- Keep ids stable when only the wording changes.
- Fill every supported locale key used by the file.
- Use arrays only for intentional multiline text.
- Run `xrf-cli translation format --path src/engine/translations` after adding records by hand.
- Run `npm run cli -- verify translations` after translation edits.
- Run `npm run cli build -- --include translations` before packaging.
- Do not patch generated files under `target/gamedata/configs/text`.
