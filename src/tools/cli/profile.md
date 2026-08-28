# Profile

`profile run` measures one invocation across one or more builds and reports what it found as a report envelope.

```powershell
xrf-cli profile run -b .\target\release\xrf-cli.exe -- gamedata verify .\target\gamedata
```

Everything after `--` is passed unchanged to every binary. The measured command's own output is discarded, so what it
prints never lands in the profiling run.

## Why it is not a stopwatch

Two correct runs of the same binary over a large tree differ by up to a second. Timing a command once, changing
something, and timing it again measures the difference between two moments as much as between two builds. The command
exists to remove the ways that goes wrong:

- **Rounds are interleaved.** With more than one `--binary`, every build runs once before any build runs twice. Running
  all rounds of one and then all of the other compares two machine states — file cache, background load, thermal
  headroom — and has produced apparent differences of 10–16% that were not real.
- **The first round is discarded.** A cold file cache costs up to twice the warm figure. `--warmup` controls how many
  rounds are thrown away; one is enough, because the second touch of the same tree is already warm.
- **The median is reported, not the mean.** A mean is dragged by the single round where something else woke up. The
  median of an even number of rounds is the lower of the two middle rounds, so every figure quoted is one that a round
  actually produced.
- **Every round is reported.** `runs` carries each measurement in execution order, so the spread is visible rather than
  hidden behind the median.

## Comparing builds

Repeat `--binary`. The first one named is the baseline, and every other reports `deltaPercent` against it — negative is
faster.

```powershell
xrf-cli profile run -b .\old\xrf-cli.exe -b .\target\release\xrf-cli.exe --rounds 5 -- gamedata verify .\target\gamedata
```

Comparisons are only ever made **within one session**. There is no saved baseline to compare against later, and that is
deliberate: a measurement taken last week against one taken today is exactly the non-interleaved comparison described
above, with more time between the halves rather than less. To compare against an older build, measure it now, beside the
new one.

Each build is identified by its own `--version` output rather than by the checkout it sits next to, because a local
build is routinely older than the source beside it and only the binary knows which commit it came from.

## Exit codes

The run succeeds when the measurement succeeded. A measured command that fails its own check does not fail the profiling
run — verifying real gamedata reports findings as a matter of course — but every exit code observed is recorded in
`exitCodes`. More than one value there means the command changed behaviour partway through the session, and its median
is describing two different pieces of work.

## Command reference

{{#include reference/profile.md:commands}}
