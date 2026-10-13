# ORCA hardware-in-the-loop results

Results from the [ORCA](https://github.com/englacial/orca) hardware-in-the-loop (HIL) tests. They run on a
Raspberry Pi 5 driving a B206mini-i SDR, with TX looped back to RX through a 30 dB pad and 50 m of
LMR-100 coax. The test definitions and harness live in `tests/hil/` of the ORCA repository.

Every push builds the site in `.github/workflows/pages.yml` and deploys it to GitHub Pages.

## Layout

```
history/history-<YYYY>.csv     one row per metric per run (append-only)
runs/<run_id>/
  run.yaml                     tier, timing, ORCA commit, overall and per-test status
  configuration.yaml           host, SDR, UHD build and fixture snapshot
  <test_id>/metrics.yaml       every metric with its bound, baseline and status
  <test_id>/*.png              full tier always; fast tier only for failed tests (or --publish-plots)
  <test_id>/reduced.zarr.zip   reduced curves, full tier only
```

`run_id` is `<UTC start>_<ORCA short sha>[-dirty]_<tier>`. Runs are published by
`python -m tests.hil.orca_hil.publish`, or at the end of a session run with `--publish`.

## History CSV

Columns, in order:

| Column | Meaning |
|---|---|
| `run_id`, `timestamp_utc` | run identifier and start time |
| `git_sha`, `dirty` | ORCA commit and whether the working tree had uncommitted changes |
| `uhd_commit`, `uhd_dirty` | UHD build stamp, so UHD changes can be told apart from ORCA changes or hardware drift |
| `tier`, `fixture_id` | `fast` or `full`; the hardware fixture |
| `test_id`, `case`, `metric` | what was measured (`test_id` is `session` for whole-run metrics) |
| `value`, `units` | the measurement; booleans are `true`/`false`, missing values are empty |
| `kind` | `abs` (physics bound), `base` (relative to the fixture baseline) or `info` (not judged) |
| `bound_type`, `bound_lo`, `bound_hi` | the bound applied in this run |
| `baseline_mean`, `baseline_std` | the baseline in force, for `base` metrics |
| `status` | `pass`, `fail`, `inconclusive` or `info` |
| `fixture_suspect` | the test flagged the hardware fixture rather than the radar |

The file is append-only: past rows are never rewritten. Adding a column is a deliberate migration and
is recorded here.

## Building the site locally

From an ORCA checkout with the `orca` environment:

```
python -m tests.hil.orca_hil.report.build_site --results <this repo> --out _site
```
