# Approved feature extraction v1

This is an exact source/output snapshot of the approved 82-screen extraction in the surrounding thesis project. It preserves the original algorithm and the selected optional correction without rewriting them. It is an archive, not a standalone dataset distribution: screenshots, metadata, the original baseline results and the 12-screen experimental fixtures remain in the main workspace.

## Source map

- [source/extract_features.py](source/extract_features.py): image loading, weighted K-Means palette/CIELAB, original foreground mask, independent Canny density, component boxes and union layout.
- [source/test_features.py](source/test_features.py): existing numerical extraction tests.
- [source/experiments/final_baseline_correction/correct.py](source/experiments/final_baseline_correction/correct.py): selected mask correction, disabled by default.
- [source/experiments/final_baseline_correction/run_pilot.py](source/experiments/final_baseline_correction/run_pilot.py): approved full-pilot runner; enables the correction and checks baseline/sample reproduction.
- [source/experiments/preprocessing_v2/](source/experiments/preprocessing_v2/): existing geometry, validation, file and rendering helpers imported by the selected runner. Their inclusion does not select the earlier row-background/local-median candidates.
- [source/requirements-lock.txt](source/requirements-lock.txt): existing dependency snapshot; reuse the project's `.venv`.

Source files are byte-identical copies with their original project-relative structure and imports. Do not run archived entry points as a standalone package: their expected datasets and historical fixtures are intentionally not duplicated here. Use the authoritative main-workspace entry points below.

## Fixed correction parameters

The saved [configuration](results_corrected_pilot/configuration.json) is authoritative. The side band width is 3.5% of processed width; qualifying side runs have adjacent RGB steps at most 6, length at least 25% of image height and RGB span at least 40. Both sides must qualify, and baseline foreground coverage must exceed .5. The candidate smooth region uses maximum four-neighbor RGB step at most 6; its area must meet the saved .35 image-area condition and it must span both side bands. Exactly one selected region is suppressed; no new foreground pixels are added. See the copied source for exact inequalities, connectivity and tie-breaking.

For notation, a processed RGB image is I with width W, height H and N=WH pixels. Pixel p has three-channel value I(p). M(p) is its baseline binary foreground mask; R(p) is the indicator of the selected smooth region (1 inside, 0 outside). When correction triggers, the revised mask is M′(p)=M(p)(1−R(p)); otherwise M′(p)=M(p). Revised coverage is F′=(Σp M′(p))/N, where Σp sums over all pixels. RGB step is the Euclidean norm of the difference between neighboring RGB vectors; span is the Euclidean norm of the three per-channel ranges over a side run. These are frozen heuristics, not semantic labels or accuracy estimates.

Color and Canny results are preserved. Only `APP02-FLOW01-S01` triggers on this pilot; 81 other screens retain baseline features. No claim is made that this correction solves all gradients, pale controls, artifact interference or segmentation failures.

## Reproduction in the existing workspace

From `/Users/kenmac/Documents/thesis-dataset`, the original approved command is:

```sh
.venv/bin/python experiments/final_baseline_correction/run_pilot.py --output results_corrected_pilot_reproduction
```

Choose a fresh output name inside the project; the runner refuses existing directories. This command would rerun extraction and its baseline/sample checks. **It was not executed for this folder update**: the existing approved results were copied and verified byte-for-byte instead. Existing numerical tests, when needed, run from the project root with `.venv/bin/python -m unittest -v test_features`.

## Snapshot verification

[snapshot_manifest.json](snapshot_manifest.json) records every copied file, original project-relative source and SHA-256 digest, plus hashes of the preserved original baseline package and `run_01` files. Copied historical provenance files may contain original absolute paths; those paths describe the original run, not this archive's location.

This update checked all 183 copied source/output files against their originals, verified 82 unique successful screen records and the saved correction activation, and confirmed that all pre-existing baseline package and `run_01` files remained unchanged. It did not rerun algorithms or introduce new numerical results. Hash equality verifies copying, not scientific validity.
