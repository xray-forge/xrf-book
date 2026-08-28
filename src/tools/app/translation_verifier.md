# Translations Verifier

The translations verifier reports which translations are missing, and from which languages. It reads the JSON sources a
project authors and never writes anything.

## What it answers

For every source and every language: how many ids does this language have text for, and how many is it still short. The
result is one row per file and language, sortable by how much is missing, with the incomplete languages named up front.

## Fields

| Field    | Purpose                                                                    |
| -------- | -------------------------------------------------------------------------- |
| Sources  | Translations directory, project root, or installation holding the sources. |
| Language | One language, or `all` for every language the build compiles.              |

Reading goes through the virtual file system, so a source tree layered over an installation is checked the way the
engine would load it rather than the way a directory walk happens to find it.

## What counts as missing

An id present with a `null` value is missing. That is exactly what a placeholder is — a gap left for a translator — so a
file full of them reads as incomplete rather than done. This is the same rule
[`translation initialize`](../cli/translation.md#initialize) writes and the [translations parser](translation_parser.md)
fills.

A complete language still gets a row, showing zero missing. A table listing only failures could not tell a finished
language apart from one the project does not carry at all.

## Counts, not names

The check reports how many ids are missing, not which. Checking a two-language import against all eight languages means
149,979 individual gaps — a correct answer, and one no table can be read from. The counts say which languages need work
and where; when the ids themselves are what you need:

```powershell
xrf-cli translation verify --path ./translations --report gaps.json
```

## Limitations

It judges completeness, not correctness: text that is present but wrong, untranslated, or copied from another language
counts as present. There is no pass or fail threshold here either —
[`translation verify --strict`](../cli/translation.md#verify) is what a build step gates on.

## CLI equivalent

[`xrf-cli translation verify`](../cli/translation.md#verify).
