# Translations command

`translations` contains helper commands for JSON translation sources and original X-Ray XML string tables.

```powershell
npm run cli -- translations <command>
```

## Commands

| Command                       | Purpose                                                                  |
| ----------------------------- | ------------------------------------------------------------------------ |
| `translations init <path>`    | Add all configured language keys to a JSON translation file or folder.   |
| `translations to_json <path>` | Convert XML string table files or folders into JSON translation sources. |
| `translations check`          | Verify the project translation sources under `src/engine/translations`.  |

The configured languages come from `cli/config.json`. Current locales are `eng`, `fra`, `ger`, `ita`, `pol`, `rus`,
`spa`, and `ukr`.

## Options

`init` supports:

- `-v, --verbose`: print verbose logs.

`to_json` supports:

- `-l, --language <locale>`: language key to fill from the XML text.
- `-c, --clean`: clean the output path before writing.
- `-o, --output <path>`: output file or directory. Defaults to `target/parsed`.
- `-e, --encoding <encoding>`: source XML encoding. If omitted, the command tries to read the XML header and then uses
  locale defaults.
- `-v, --verbose`: print verbose logs.

`check` supports:

- `-l, --language <locale>`: check one language instead of all languages.
- `-s, --strict`: fail on missing entries.
- `-v, --verbose`: print verbose logs.

## Examples

```powershell
npm run cli -- translations init src/engine/translations/st_dialogs.json
npm run cli -- translations to_json configs/text/eng --language eng --output src/engine/translations
npm run cli -- translations check
npm run cli -- translations check --language ukr --strict
```

## Failure notes

`to_json` requires a known locale. When converting a folder, the output path must be a directory, not a single file.
