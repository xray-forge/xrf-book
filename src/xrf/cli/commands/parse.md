# Parse

`parse` contains utility commands that generate JSON or HTML support files under `target/parsed`.

```powershell
npm run cli -- parse <command>
```

## Commands

| Command                    | Purpose                                                                     | Output                        |
| -------------------------- | --------------------------------------------------------------------------- | ----------------------------- |
| `parse dir_as_json <path>` | Flatten a directory tree into a JSON object keyed by normalized file names. | `target/parsed/<folder>.json` |

`dir_as_json` resolves `<path>` relative to the repository root.

## Options

`parse dir_as_json` supports:

- `-e, --no-extension`: omit file extensions in JSON values.

## Examples

```powershell
npm run cli -- parse dir_as_json src/resources/textures
npm run cli -- parse dir_as_json src/resources/textures --no-extension
```

## Output usage

Use `dir_as_json` when another script needs a compact index of files under a resource folder. The command writes
generated support data under `target/parsed`, so treat the result as disposable build output.

Use `xrf-cli externs export` when checking the script declaration surface exposed by conditions, effects, and dialogs.

## Failure notes

`dir_as_json` requires a path argument.
