# Extern exports

Use `xrf-cli externs export` to generate or check a manifest of script exports declared through TypeScript `extern(...)`
calls. Choose JSON for the tracked contract, XML for structured export, or HTML for a browsable reference.

## Export a manifest

The examples below run from the `xrf-engine` repository root and require `xrf-cli` on your executable search path. For
another declaration tree, replace `src/engine/declarations` with its root.

This command creates or replaces `target/parsed/externs.html`, creating parent directories when needed:

```powershell
xrf-cli externs export --path src/engine/declarations `
  --dest target/parsed/externs.html
```

Example output — export declarations:

```text
Exported 3 externs to './extern.json'.
```

Exit code: `0`.

Exit code `0` means the export succeeded. Open the resulting HTML file to browse namespaces and declarations. Manifest
source paths are relative to the declarations root.

## Formats

| Format | Artifact                           | Default line endings |
| ------ | ---------------------------------- | -------------------- |
| `json` | Manifest with an `exports` object. | CRLF                 |
| `xml`  | `<externs><exports>` document.     | LF                   |
| `html` | Collapsible namespace reference.   | LF                   |

`-p, --path` names the declarations root and `-d, --dest` names the artifact. The format is inferred from the artifact's
extension; `--format json`, `--format xml`, or `--format html` overrides it, for example for an artifact with another
extension. Use `--line-endings lf` or `--line-endings crlf` to override the format's default.

## Check

Add `--check` to compare the existing `--dest` artifact with the declarations without writing either:

```powershell
xrf-cli externs export --path src/engine/declarations `
  --dest src/engine/declarations/extern.json --check
```

The format is chosen the same way as when writing. JSON is compared as parsed manifest data; XML and HTML are compared
as rendered text with line-ending differences ignored. This check does not enforce a line-ending policy.

Exit code `0` means the artifact matches. Exit code `3` indicates a mismatch or invalid declaration/artifact content;
inspect the diagnostic before regenerating. An input that cannot be read is an execution failure, exit code `1`. See
[CLI reporting and exit codes](cli.md#exit-codes) for the shared contract.

The engine wrapper for this check is `npm run cli -- verify externs`. It is available explicitly and is not a build or
CI gate.

## Requirements

Export names must be unique string literals. The parser reads supported function and value references from their
declared TypeScript contracts; use an explicit `value as Type` assertion when a value needs its export type stated.
Missing or unrenderable callable types are emitted as `unknown`. Unsupported declarations produce diagnostics rather
than inferred runtime contracts.

The command skips `*.test.ts`, `*.spec.ts`, and sources under `__test__`.

## Command reference

{{#include reference/externs.md:commands}}
