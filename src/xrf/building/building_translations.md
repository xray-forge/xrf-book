# Building Translations

The translations target writes game string tables to `target/gamedata/configs/text` from `src/engine/translations`
through the bundled XRF tools binary.

## Build translations

```powershell
npm run cli -- build --include translations
npm run cli -- build -i translations
```

## Selecting a Language

The default locale is configured in `cli/config.json` as `locale`. The current default is `ukr`.

Override it for a build with `--language`:

```powershell
npm run cli -- build --include translations --language eng
```

Supported locale keys are listed in `cli/config.json` under `available_locales`.

## Locale Resource Packs

Resource packs for voice and localized assets are configured under `resources.mod_assets_locales` in `cli/config.json`.
When asset overrides are enabled, the resources build includes override folders plus the locale folders for the selected
language.

## Sources

XRF uses JSON translation sources for generated multilingual output. Check them with
`npm run cli -- verify translations`; see [Translations](../../script_engine/translations.md).
