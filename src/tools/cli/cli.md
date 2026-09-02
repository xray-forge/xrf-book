# Tools CLI

The Rust `xrf-cli` binary provides repeatable asset inspection, conversion, packing, unpacking, formatting, and
verification outside the engine repository wrapper.

```powershell
xrf-cli <group> <command> --help
```

Every command belongs to a group, so operations read as `xrf-cli archive pack` or `xrf-cli gamedata verify`. The engine
CLI wraps selected operations through `npm run cli -- ...`; use `xrf-cli` directly for scripts and format-specific
workflows.

## Command groups

{{#include reference/README.md:groups}}

## Reporting

Four options are available on every command.

| Option            | Effect                                                                            |
| ----------------- | --------------------------------------------------------------------------------- |
| `-s, --silent`    | Say nothing but the fact that a run failed.                                       |
| `-v, --verbose`   | Say more than a normal run does.                                                  |
| `--json`          | Write one JSON report to standard output and move human output to standard error. |
| `--report <PATH>` | Write the same JSON report to a file, leaving human output alone.                 |

`--silent` and `--verbose` cannot be combined, and neither can `--json` and `--report`; either pair is a usage error.
Rust logging honours `RUST_LOG` when it is set.

`--json` is what a script or an agent should use: standard output carries exactly one JSON document and nothing else, so
it can be piped straight into a parser.

```powershell
xrf-cli gamedata verify .\target\gamedata --json | ConvertFrom-Json
```

The document is the same in either mode:

```json
{
  "build": {
    "version": "0.1.0",
    "kind": "optimized",
    "commit": "791cd5014b9fa842e3e47419e22dcce023474784",
    "reference": "main",
    "isDirty": false,
    "builtAt": "2026-08-28T09:14:02Z",
    "target": "x86_64-pc-windows-msvc",
    "rustc": "rustc 1.97.1",
    "profile": "release",
    "optimization": "opt-level=s, lto=true, codegen-units=1",
    "runId": null
  },
  "command": ["gamedata", "verify"],
  "duration": 1204,
  "error": null,
  "execution": { "workers": 8, "origin": "auto" },
  "exitCode": 0,
  "outcome": "success",
  "result": {}
}
```

`outcome` is `success`, `checkFailed`, or `executionFailed`, and `exitCode` is the code the process then exits with.
`result` carries whatever the command found, in that command's own shape, and is `null` for a command that reports no
structured result yet. A failing run still produces the document, so a check that judged its input invalid reports the
findings explaining the verdict rather than only a non-zero exit.

`build` says which binary produced the document. A report outlives the run that wrote it, so comparing two of them —
across releases, or against a result someone else reported — means knowing what each was measured with. `isDirty`
separates a report from a released commit from one produced by a working tree that merely started at that commit, and
`runId` names the workflow run for a binary that came from CI rather than a developer's machine.

`execution` says how wide the run was, for the same reason and on every report: comparing two of them means knowing how
much of each machine was used. See below.

## Execution

Commands with work to spread take `-j, --jobs`. It is not on every command, because a command with no parallel work has
nothing to bound.

| Value        | Meaning                                                       |
| ------------ | ------------------------------------------------------------- |
| `auto`       | Whatever the machine offers. The default when `-j` is absent. |
| `<count>`    | Exactly that many workers. `1` is a real sequential run.      |
| `<percent>%` | That share of the machine, rounded down, never below one.     |

```powershell
xrf-cli gamedata verify .\target\gamedata -j 50%
```

## Exit codes

| Code | Meaning                                             |
| ---- | --------------------------------------------------- |
| 0    | Success.                                            |
| 1    | The command could not do its job.                   |
| 2    | The invocation was rejected before the command ran. |
| 3    | A check ran and judged its input invalid.           |

Only `verify` commands and `--check` or `--strict` modes answer 3, and only for content they judged. An unreadable file,
a refused write, or a check that reached no verdict answers 1. A requested report that cannot be written also answers 1,
whatever the command itself decided, so a script never reads a previous run's report as this run's answer.

## Command reference

Each group page carries its own guidance first and a generated command reference after it. The reference comes from the
command definitions themselves, so an option added to the tool reaches this book without anyone rewriting a table:

```powershell
npm run cli:reference
npm run format
```

The first regenerates `src/tools/cli/reference/` and needs the `xrf-tools` repository checked out beside this one; the
second brings the new pages into this book's formatting. Those pages are never edited by hand: a correction to an
option's description belongs in the command definition it came from.
