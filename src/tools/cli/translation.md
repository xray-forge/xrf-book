# Translation CLI

Translation commands work with XRF JSON translation projects and generated gamedata string tables.

## Initialize

`translation initialize` ensures translation files carry the expected language keys, adding the missing ones as nulls.
Running it twice changes nothing the second time.

```powershell
xrf-cli translation initialize --path ./translations
```

## Format

`translation format` normalizes JSON sources: ids and language keys sorted, two-space indentation, a trailing newline.
`--check` reports what is not normalized without writing anything and exits non-zero, which is what a build step or a
pre-commit hook gates on.

```powershell
xrf-cli translation format --path ./translations
xrf-cli translation format --path ./translations --check
```

`--path` is repeatable and takes files or folders. A folder is walked for JSON sources; a file named directly is taken
whatever its extension, so a source with an unusual name can still be formatted, and it fails loudly if it turns out not
to be one.

Sorting is natural rather than by byte, matching how rustfmt orders identifiers: `st_thanks2` comes before
`st_thanks10`, and `ammo-5.45x39-ap` before `ammo-11.43x23-fmj`. Byte order puts the longer number first, which reads
wrong in every review that touches one.

Line endings are left alone. Each file keeps the convention it already has, because that belongs to `.gitattributes`
rather than to a formatter, and a check does not judge them. `--line-endings lf|crlf` overrides that and arms the check
on it, so a project that wants one spelling enforced can ask for it.

### What it does not touch

Values are never rewritten. A one-element `["text"]` stays an array and a string holding a literal `\n` stays a string:
the build joins the array form on that same literal, so both ship byte-identical text to the game, and choosing between
them is an authoring decision rather than a formatting one. No `null` placeholder is added either - that is what
`initialize` and `parse` do, and `verify` already counts an absent language and an explicit `null` as the same gap.

A source already holding the canonical bytes is not rewritten at all, so a run over a clean tree changes no timestamps.

### When it refuses

Two cases exit non-zero without reaching a verdict, rather than reporting a finding. A set of paths that selects no
sources is refused, so a renamed folder cannot quietly make a check gate pass over nothing. A source that cannot be
parsed stops the run at that file - a formatter has nothing to write except what it read, and rewriting a file it failed
to understand is how work gets destroyed. Sources already rewritten stay rewritten; each is replaced whole, so nothing
is left half-written.

## Build

`translation build` compiles translation JSON into gamedata string tables, one file per source per language, each
written in that language's code page.

```powershell
xrf-cli translation build --path ./translations --output ./gamedata/configs/text --language ukr
```

`--path` names roots and reads through the virtual file system, so a source tree layered over an installation compiles
what the engine would load. One source file is still accepted directly. The output is always a plain directory — a
string table is a file, and an archive has nowhere to put one — and it may not sit inside any of the source roots.

A missing translation compiles to the id itself, which is the engine's own fallback, so every language gets a complete
table rather than a short one. The report carries a row per language with the tables written and ids compiled.

## Verify

`translation verify` checks completeness: every id a language has no text for, counting an explicit `null` placeholder
as missing. Gaps are reported as findings; `--strict` is what turns them into a non-zero exit, which is what a build
pipeline gates on.

```powershell
xrf-cli translation verify --path ./translations --language ukr --strict
```

`--path` names roots and reads through the virtual file system, so a source tree layered over an installation is checked
the way the engine would load it. One source file is still accepted directly, since a single file needs no roots.

The report carries two things: a finding per missing id, and a `languages` array with one row per file and language. The
rows are what make the report readable at scale — checking a two-language import against all eight languages produces
149,979 findings, and the same answer is 1,072 rows.

## Parse

`translation parse` imports raw X-Ray XML string tables into JSON sources. It is the way a downloaded mod, a gamedata
tree, or an installed game becomes something the other three commands can work with.

One run reads one language. Raw XML carries no language of its own, so `--language` declares it rather than the tool
guessing — a tree read under the wrong key files every string it holds under the wrong language, with nothing afterwards
to say so.

```powershell
xrf-cli translation parse --path ./stalker-anomaly --language eng --output ./translations
xrf-cli translation parse --path ./stalker-anomaly --language ukr --output ./translations
```

Run it once per language into the same output and the languages collect into one JSON per table. Reading is through the
virtual file system, so an installation whose tables live inside `db\configs` imports exactly like a loose folder.

### What it writes

Sources are named after the tables they came from, mirroring any subdirectories. Ids and language keys are sorted into
the same canonical order [`format`](#format) produces, so the result does not depend on which language you ran first,
and re-running changes nothing. A record missing one of the languages its file carries gets an explicit `null`, which is
what makes the gap visible to a translator.

Text already in the output is kept when it differs from what was read; the run reports how many entries diverged.
`--overwrite` is what replaces them.

### Finding the tables

`--path` names a root, not the text directory. The run looks under `configs\text` when the root has one, then descends
into the directory named for the language. `--prefix` overrides that when a tree is laid out differently. A scope that
still holds another language's directory is refused rather than read, since reading it would file those strings under
the wrong language.

### Before writing anything

`--dry-run` reports exactly what a run would change without writing it. `--file` narrows a run to one table, and
`--strict` turns anything unreadable into a non-zero exit for a build step.

## Notes

The engine repository wraps `translation verify` as `npm run cli -- verify translations` and `translation build` as the
`translations` build target. It has no wrapper for `format`, `initialize` or `parse`; call `xrf-cli` directly for those.

## Command reference

{{#include reference/translation.md:commands}}
