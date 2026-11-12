# Approved workflow and project map

Paths below are relative to the located thesis project, not to this installed skill.

## Authoritative artifacts

| Path | Role |
|---|---|
| `extract_features.py`, `test_features.py` | Original baseline implementation and numerical tests |
| `requirements-lock.txt`, `.venv/` | Existing dependency snapshot and environment |
| `PILOT/screenshots/screens.csv` | 82-screen metadata; UTF-8 BOM; screenshot paths relative to this CSV |
| `PILOT/screenshots/flows.csv` | Application/flow metadata; not used to score extraction |
| `results/` | Original saved extraction baseline; also contains a separate downstream composite directory |
| `experiments/final_baseline_correction/correct.py` | Selected optional foreground correction |
| `experiments/final_baseline_correction/run_01/` | Frozen 12-screen sample evaluation and candidate outputs |
| `experiments/final_baseline_correction/run_pilot.py` | Selected method applied to the full approved pilot |
| `results_corrected_pilot/` | Approved 82-screen extraction outputs |
| `experiments/preprocessing_v2/revised_pipeline.py`, `review_assets.py` | Existing imported geometry, validation, I/O and rendering helpers |
| `visual_quality_review/selected_screens.csv` | Fixed 12-screen review sample |
| `Feature Extraction/approved_feature_extraction_v1/` | Source/output archive, with SHA-256 snapshot manifest |
| `Feature Extraction/run_01/` | Preserved copy of the sample evidence, not the full 82-screen run |
| `Feature Extraction/feature_extraction_github/` | Preserved original baseline code package |

The approved archive is not a standalone executable package. Its copied scripts retain original imports and expect the main project's datasets and fixtures. Use the main-workspace entry points. Do not rewrite archive provenance paths as though it were a new extraction run.

## Commands and prerequisites

Run from the project root. Inspect `--help` and the entry point before execution if the source has changed.

For the baseline's existing tests:

```sh
.venv/bin/python -m unittest -v test_features
```

The tests decode original images and run synthetic extraction checks; they are not necessary for copying existing artifacts.

For the original baseline on a small selection:

```sh
.venv/bin/python extract_features.py --output results/baseline_smoke_new --screen-ids APP01-FLOW01-S00 APP01-FLOW01-S02 APP01-FLOW02-S04 --plots
```

For an explicitly requested reproduction of the approved full pilot:

```sh
.venv/bin/python experiments/final_baseline_correction/run_pilot.py --output results_corrected_pilot_reproduction
```

Choose a fresh name if that destination exists. The full runner requires the original `results` baseline plus the frozen 12-screen candidate fixtures. It asserts exactly 82 distinct pilot IDs, reproduces original features, repeats the correction to check determinism, checks unchanged masks when the correction abstains, and compares relevant screens against the frozen sample. It records errors and returns a failing exit status if extraction failed. A numerical/provenance assertion failure requires investigation, not deleting the check or presenting a partial run as complete.

The script's `--output` must remain in the project. Missing input fixtures or dependencies are blockers for that command; identify the missing prerequisite and continue read-only inspection where possible. Do not install replacement versions or regenerate missing historical results silently.

## Expected approved result

The recorded approved run has 82 successes and zero errors. Correction activates only for `APP02-FLOW01-S01`; the other 81 screens retain baseline features. This is a check against this specific pilot, not a hardcoded selection rule and not an expected activation count for another dataset. Pay particular attention to `APP05-FLOW01-S04` as well when visually assessing inherited mixed-background/pale-control limitations.

Separate three claims:

1. **Numerical correctness:** valid records, formulas, finite bounded fractions and aligned IDs.
2. **Reproducibility:** matching saved numbers/configuration under the fixed environment and unchanged protected files.
3. **Visual quality:** actual inspection of segmentation, missed edges, fragmentation, filled regions and artifact influence.

Passing the first two does not prove the third. Review any newly activated screen and report regressions rather than relying on foreground coverage alone. Do not start a full-data optimization experiment merely because a user asked to inspect existing results.

For archive maintenance, compare copied bytes against the authoritative source/output files and maintain `snapshot_manifest.json`. Preserve the original baseline package, `run_01`, and prior versions. Git commits or GitHub uploads require their own user instruction; creating or updating a local archive does not imply publication.
