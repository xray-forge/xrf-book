# Translations Parser

The translations parser imports raw X-Ray XML string tables into the JSON sources the rest of the tooling reads. It is
how a downloaded mod, a gamedata tree, or an installed game becomes a translation project.

## What it is for

XRF authors translations as multi-language JSON, one file per string table, every language keyed inside it. Anything
shipped by the game or by a mod is XML instead, split one directory per language. The parser converts the second into
the first, and merges each run into whatever is already there — so importing English and then Ukrainian leaves one file
per table carrying both.

## One language per run

Raw XML carries no language of its own. The directory holding it usually does, but a mod can lay its files out however
it likes, so the language is declared rather than guessed: reading a tree under the wrong key files every string it
holds under the wrong language, and nothing afterwards can tell.

Pick the language in the form and run it once per language you want to collect.

## Fields

| Field         | Purpose                                                                            |
| ------------- | ---------------------------------------------------------------------------------- |
| Source        | Mod folder, gamedata tree, or game installation holding the XML string tables.     |
| Language      | The language every entry this run reads is filed under.                            |
| Output        | Directory the JSON sources are written to, merging with any already there.         |
| Existing text | Whether text already in the output is replaced when it differs from what was read. |

The source names a **root**, not the text directory. The run looks under `configs\text` when the root has one and then
descends into the directory named for the language, so pointing at the top of a mod is enough. Reading goes through the
virtual file system, which is why an installed game works: on Anomaly and CoC the tables live inside `db\configs`, where
a plain folder reader finds nothing at all.

A source still holding another language's directory is refused rather than read, since reading it would file those
strings under the language you named.

## Preview before writing

**Preview** runs the import without writing anything and reports exactly what it would change — files created, entries
inserted, placeholders added, and how many entries differ from text already in the output. **Import** does the same and
writes.

## Reading the result

| Count        | Meaning                                                             |
| ------------ | ------------------------------------------------------------------- |
| files read   | String tables read out of the source.                               |
| created      | Sources that did not exist yet.                                     |
| updated      | Sources merging changed something in.                               |
| unchanged    | Sources merging changed nothing in — a re-run reports all of these. |
| inserted     | Ids new to their file.                                              |
| filled       | Placeholders replaced with text.                                    |
| placeholders | `null`s added for languages a file carries but a record lacked.     |
| conflicts    | Entries whose existing text differed from what was read.            |

Conflicts are not failures. Text already in a source is usually a translator's own work, so the import keeps it and
reports the count; tick **Replace existing text that differs** and run again to take the imported wording instead.

The findings table lists anything that could not be read — a malformed table, a document that is not a string table, or
an id defined twice in one file. Each costs its own strings and nothing else, so one bad file never stops an import.

## Afterwards

The imported sources are an ordinary translation project: open them in the [translation editor](translation_editor.md),
scaffold the remaining languages with `translation initialize`, check them with `translation verify`, and compile them
with `translation build`.

## CLI equivalent

[`xrf-cli translation parse`](../cli/translation.md#parse) does the same work, and is what a scripted import uses.
