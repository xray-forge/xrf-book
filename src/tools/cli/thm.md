# THM CLI

`thm patch-bump` changes the bump texture declared by a `.thm` descriptor.

```powershell
xrf-cli thm patch-bump --path ./textures/wpn/wpn_pm/wpn_pm.thm --to "wpn\wpn_pm\wpn_pm_bump" --dry-run
```

Remove `--dry-run` to write the change. Use `--dest` to write a separate descriptor, or `--off` instead of `--to` to
disable the bump declaration. See [Bump declarations](dds.md#bump-declarations) for the workflow and engine behavior.

## Command reference

{{#include reference/thm.md:commands}}
