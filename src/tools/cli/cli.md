# Tools CLI

The Rust `xrf-cli` binary inspects, converts, packs, and verifies X-Ray assets. Use it directly for asset workflows and
automation; the engine repository's `npm run cli -- ...` wrapper exposes selected operations.

Commands have a group and an operation. To inspect a command's accepted arguments:

```powershell
xrf-cli archive pack --help
```

Examples in this chapter assume `xrf-cli` is on `PATH`. Relative paths resolve from the current directory; each workflow
identifies its input layout. Output blocks show demo runs; paths, counts, and timings depend on the input and machine.

## Command groups

{{#include reference/README.md:groups}}

## Reporting

| Option            | Effect                                                            |
| ----------------- | ----------------------------------------------------------------- |
| `-s, --silent`    | Suppress ordinary logging; a failed run still reports failure.    |
| `-v, --verbose`   | Include command-specific detail.                                  |
| `--json`          | Write one JSON report to stdout and human output to stderr.       |
| `--report <PATH>` | Write the same JSON report to a file; human output stays enabled. |

`--silent` conflicts with `--verbose`; `--json` conflicts with `--report`. Rust logging also honors `RUST_LOG`.

Prefer a report file for large verification runs. Capture the exit code immediately after the command, then read the
fields needed for the decision. From a project containing `target/gamedata`:

```powershell
xrf-cli gamedata verify ./target/gamedata --report ./verification-report.json 2>$null
$verificationExit = $LASTEXITCODE
$report = Get-Content ./verification-report.json -Raw | ConvertFrom-Json
$report.result.status
$verificationExit
```

Use `--json` when a consumer needs a stdout pipe. Neither report mode limits the number of findings.

The report is an envelope around a command-specific `result`:

| Field       | Meaning                                                                                  |
| ----------- | ---------------------------------------------------------------------------------------- |
| `build`     | Binary version, commit, build settings, dirty state, and CI run identity when available. |
| `command`   | Group and operation names.                                                               |
| `duration`  | Total duration in whole milliseconds.                                                    |
| `execution` | Worker count and how it was selected.                                                    |
| `exitCode`  | Command exit code.                                                                       |
| `outcome`   | `success`, `checkFailed`, or `executionFailed`.                                          |
| `error`     | Failure details, or `null` on success.                                                   |
| `result`    | The command's structured answer; it is `null` when no structured answer was produced.    |

A failed check still reports its findings. Argument parsing failures occur before command execution and do not produce
an envelope. If writing the report fails, the process exits 1; an existing report at that path may belong to an earlier
run. Check the process result and report freshness before using a saved answer.

Keep `build` and `execution` when comparing reports: different binaries or worker counts can explain different results
and timings.

## Execution

Commands that support parallel work accept `-j, --jobs`:

| Value            | Meaning                                                                             |
| ---------------- | ----------------------------------------------------------------------------------- |
| `auto`           | Use the machine's available parallelism; the default.                               |
| A positive count | Use that many workers. `1` runs sequentially.                                       |
| A percentage     | Use that share of available parallelism, rounded down with a minimum of one worker. |

```powershell
xrf-cli gamedata verify ./target/gamedata -j 50%
```

Commands without parallel work do not accept `--jobs`.

## Exit codes

| Code | Meaning                                                              |
| ---- | -------------------------------------------------------------------- |
| 0    | The command succeeded.                                               |
| 1    | Execution failed or verification could not reach a complete verdict. |
| 2    | The invocation was rejected before the command ran.                  |
| 3    | A check ran and judged its input invalid.                            |

A command's `--strict` behavior is specific to that command. Consult its guide before treating every strict failure as
exit 3; refused writes and execution errors still use exit 1.

## Command reference

Each group page combines authored workflow guidance with reference generated from the command definitions. Correct
option descriptions in the source command, then follow [the reference-generation workflow](docs.md).
