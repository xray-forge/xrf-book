# Translation CLI

Translation commands work with XRF JSON translation projects and generated gamedata string tables.

## Initialize

`translation initialize` ensures translation files carry the expected language keys, adding the missing ones as nulls.
Running it twice changes nothing the second time.

```powershell
xrf-cli translation initialize --path ./translations
```

## Build

`translation build` compiles translation JSON into gamedata string tables, one file per language.

```powershell
xrf-cli translation build --path ./translations --output ./gamedata/configs/text --language ukr
```

## Verify

`translation verify` checks completeness. Gaps are reported as findings; `--strict` is what turns them into a non-zero
exit, which is what a build pipeline gates on.

```powershell
xrf-cli translation verify --path ./translations --language ukr --strict
```

## Parse

`translation parse` is a placeholder. It accepts `--path` but does not currently parse or write translations.

## Notes

Use the engine repository's `npm run cli -- translations ...` workflow to convert original X-Ray XML string tables to
XRF JSON sources.

## Command reference

{{#include reference/translation.md:commands}}
