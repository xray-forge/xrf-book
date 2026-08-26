# Extern exports

`externs export` reads TypeScript `extern(...)` declarations.

```powershell
xrf-cli externs export <declarations-root> --format json --output <path>
```

Manifest source paths are relative to the declarations root.

## Formats

- `json`: tracked `{ "exports": ... }` contract; CRLF by default.
- `xml`: `<externs><exports>` document; LF by default.
- `html`: collapsed namespace reference; LF by default.

Use `--line-endings lf|crlf` to override the default.

```powershell
xrf-cli externs export src/engine/declarations --format html --output target/parsed/externs.html
```

## Check

`--check <artifact>` validates without writing and cannot be combined with `--output`.

```powershell
xrf-cli externs export src/engine/declarations --check src/engine/declarations/extern.json
```

The format is inferred from the extension unless `--format` is provided. JSON is compared semantically; XML and HTML are
compared as rendered text with line-ending differences ignored.

The engine wrapper is `npm run cli -- verify externs`. It is not a build or CI gate.

## Requirements

Names must be unique string literals. Missing or unrenderable callable types are emitted as `unknown`. Values need
`value as Type`. The command skips `*.test.ts`, `*.spec.ts`, and `__test__` sources.

## Command reference

{{#include reference/externs.md:commands}}
