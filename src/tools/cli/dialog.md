# Dialog CLI

`dialog info` reads dialog XML and reports its contents and findings.

```powershell
xrf-cli dialog info --path ./gamedata --prefix configs/gameplay
```

Repeat `--path` to layer roots, highest priority first. Use `--source directory` to read a loose folder without
searching for a containing installation. Add `--strict` to fail when dialog data is unreadable or does not match the
schema.

For authoring dialog phrases and callbacks, see [Dialog configs](../../script_engine/configs_dialogs.md).

## Command reference

{{#include reference/dialog.md:commands}}
