# Docs CLI

`docs generate` creates Markdown reference pages from the CLI's command definitions. Use it to publish command names,
arguments, defaults, and help text without maintaining a second option catalog.

## Generate standalone reference

Run from a directory where `./cli-reference` can hold generated files exclusively. Generation replaces expected pages
and removes unexpected top-level `.md` files in that directory.

```powershell
xrf-cli docs generate --output ./cli-reference
xrf-cli docs generate --output ./cli-reference --check
```

The first command writes an index and group pages. The second performs a read-only comparison against what the current
binary would generate: exit 0 means they match, and exit 3 means a page is missing, outdated, or unexpected. Comparison
normalizes CRLF to LF.

Use the same binary for generation and checking. A previously built binary may describe older commands than the source
checkout beside it.

## Update the book reference

Run from the `xrf-book` repository with `xrf-tools` checked out beside it:

```powershell
npm run cli:reference
npm run format
```

The first command regenerates `src/tools/cli/reference/` from the sibling tools repository; the second applies book
formatting. Authored group pages include generated command sections from that directory.

Edit command definitions to correct reference text. Keep workflow explanations in the authored pages outside
`reference/`. Do not use `docs generate --check` against the book's formatted output: formatting changes the generated
text beyond the line-ending normalization that the check accepts.

## Command reference

{{#include reference/docs.md:commands}}
