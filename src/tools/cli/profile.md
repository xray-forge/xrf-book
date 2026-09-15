# Profile

`profile run` measures repeated executions of a command and compares builds within the same session. It reports
wall-clock duration and sampled memory use.

## Measure a command

From an `xrf-tools` checkout with a release binary and an assembled `target/gamedata` tree:

```powershell
xrf-cli profile run -b ./target/release/xrf-cli.exe --report ./profile-report.json `
  -- gamedata verify ./target/gamedata
```

Example output — profile a command:

```text
Profiling dds info --path ./gamedata/textures/ui/ui_test_sheet.dds
1 binaries, 3 rounds after 1 warmup, interleaved
round 1/3 xrf-cli: 10 ms
round 2/3 xrf-cli: 9 ms
round 3/3 xrf-cli: 11 ms
xrf-cli: median 10 ms, peak 11.8 MB / mean 11.8 MB
```

Exit code: `0`. Timings vary between runs.

Everything after `--` is passed unchanged to the measured binary. Relative paths use the current directory. The child's
stdout and stderr are discarded; measurements are returned in the profiling command's report.

The default is one warmup round followed by five measured rounds. Every invocation performs the command's normal side
effects, including warmups. Use a read-only command for a repeatable comparison, or restore its inputs between sessions.

## Comparing builds

Repeat `--binary` to compare builds. In this example, `old/xrf-cli.exe` is a previously saved build and
`target/release/xrf-cli.exe` is the candidate:

```powershell
xrf-cli profile run -b ./old/xrf-cli.exe -b ./target/release/xrf-cli.exe --rounds 5 `
  --report ./comparison.json -- gamedata verify ./target/gamedata
```

The first binary is the baseline. Other binaries report `deltaPercent` against its median duration; negative values mean
faster execution. Each binary is identified by its own `--version` output, independently of the adjacent checkout.

Use the same corpus, arguments, worker settings, and machine for all builds. Compare old and new binaries in one
session: this command does not load a historical baseline.

## Why it is not a stopwatch

Rounds are interleaved: each binary runs once, in the supplied order, before the next round starts. This limits the
effect of changing file caches, background load, and thermal conditions, but does not eliminate measurement noise.

`--warmup` controls how many initial rounds are discarded. Increase it when the workload needs more time to stabilize;
one warmup does not guarantee a warm or steady system.

The summary uses medians. With an even number of rounds, it selects the lower middle value. Individual measurements
remain in `runs` in execution order, so inspect their spread before attributing a small difference to a code change.

## Memory

Memory is sampled about every 20 ms for the measured child process, excluding its descendants:

- `peakBytes` is the largest observed resident set in a round.
- `meanBytes` is the average resident set across that round's samples.
- Summary values are independent medians of the per-round measurements.

Short-lived allocations can fall between samples. A missing measurement means no usable sample was collected, not zero
memory use. These are resident-memory figures, not total allocations or CPU utilization.

A high peak with a lower mean suggests transient memory use; similar values suggest sustained residency. Neither alone
proves whether the program released a particular allocation.

## Exit codes

Profiling succeeds when measurement succeeds, even if a measured command returns a failure code. Inspect each build's
`exitCodes` before comparing its timing: a fast failure may have done less work. Multiple observed codes mean the
command's outcome varied during the session.

## Command reference

{{#include reference/profile.md:commands}}
