# CLI documentation generation

`docs generate` writes Markdown reference pages from the CLI command definitions.

```powershell
xrf-cli docs generate --output ./target/cli-reference
xrf-cli docs generate --output ./target/cli-reference --check
```

`--check` compares an existing reference with freshly rendered pages without writing. Use it on unmodified generator
output; formatting the files changes what it compares.

To refresh this book from a sibling `xrf-tools` checkout, run these commands from `xrf-book`:

```powershell
npm run cli:reference
npm run format
```

Edit command definitions in `xrf-tools/bin/xrf-cli/src/commands` to correct generated descriptions.

## Command reference

{{#include reference/docs.md:commands}}
