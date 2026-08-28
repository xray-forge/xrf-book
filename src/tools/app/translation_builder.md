# Translations Builder

The translations builder compiles a project's JSON sources into the string tables the game loads. It is the last step
before packaging, and the only translations tool that produces files the engine reads directly.

## What it writes

One XML table per source per language, at `<output>/<language>/<name>.xml`, each encoded in that language's own code
page. Building `st_items.json` for every language writes eight files; a project of 34 sources writes 272.

## Fields

| Field    | Purpose                                                                   |
| -------- | ------------------------------------------------------------------------- |
| Sources  | Translations directory, project root, or installation holding the JSON.   |
| Language | One language, or `all` for every language the build supports.             |
| Output   | Directory the tables are written to, as `<output>/<language>/<name>.xml`. |
| Sort ids | On sorts ids; off preserves the order each source declares them in.       |

Sources are read through the virtual file system, so a tree layered over an installation compiles what the engine would
load. The output is always a plain directory, and it may not sit inside any of the source roots — a build cannot fill an
authored tree with generated files.

## Missing translations still build

An id a language has no text for compiles to the id itself, which is the engine's own fallback. Every language therefore
gets a complete table rather than a short one, and an untranslated string shows its key in game rather than nothing.
That means **a clean build is not evidence the translations are complete** — the
[translations verifier](translation_verifier.md) is what answers that.

## Encodings

Each language is written in its own code page: windows-1251 for Russian and Ukrainian, windows-1250 for German and
Polish, windows-1252 for the rest. A character the target code page cannot represent fails the build, naming the id and
the character, rather than being silently written as a replacement.

## Reading the result

A row per language, with the tables written and the ids compiled into them. Two sources that would write the same table
fail the build before anything is written, rather than letting one overwrite the other.

## Limitations

Existing tables at the destination are overwritten without asking, and nothing already there is removed first — an older
build of sources you have since renamed stays mixed in with this one. There is no progress and no cancellation; the
first failure ends the run and whatever reached disk before it stays there.

## CLI equivalent

[`xrf-cli translation build`](../cli/translation.md#build).
