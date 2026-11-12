---
name: thesis-visual-feature-extraction
description: Run, reproduce, document, or visually review the thesis mobile UI feature extraction pipeline using its approved baseline and optional foreground correction. Use for screenshot extraction and extraction-result maintenance; pairwise/composite similarity and graph scoring are separate stages.
---

# Thesis visual feature extraction

Use the existing implementation rather than rebuilding the algorithms. This skill guides the thesis pipeline; it does not bundle its dataset or Python dependencies.

## Locate the project and requested operation

The current workspace is `/Users/kenmac/Documents/thesis-dataset`. If it moved, locate `extract_features.py`, `experiments/final_baseline_correction/run_pilot.py`, and `PILOT/screenshots/screens.csv` together. Inspect local instructions and source before running. Use the existing `.venv` from the project root.

Distinguish these operations:

- **Read/review/document existing results:** load saved records and diagnostic figures. Do not rerun extraction just to summarize, copy, or package them.
- **Reproduce the approved pilot:** use `experiments/final_baseline_correction/run_pilot.py` with a fresh output directory. Read [references/workflow.md](references/workflow.md) for prerequisites, checks and commands.
- **Run the uncorrected baseline:** use root `extract_features.py`, explicitly identify the run as uncorrected, and inspect its CLI before selecting a subset or plotting.
- **Change an algorithm or use a new dataset:** treat this as a separate requested experiment. The approved full-pilot runner is tied to 82 screens and historical fixtures; do not remove its assertions to accommodate a different dataset. Keep comparisons separate from the approved results.

## Preserve the research contract

Keep original screenshots, metadata, baseline records, historical experiments and prior output directories intact. Never silently overwrite a run. User-requested intentional code changes belong to the task's stated scope; this skill does not forbid them or require another approval.

Preserve definitions unless the user explicitly requests a methodological change: full-image aspect-preserving preprocessing; weighted RGB K-Means with maximum K=5 and CIELAB output; foreground fraction; independent blurred-grayscale Canny fraction; connected-component boxes; four-column/eight-row bounding-box-union occupancy. Read [references/features.md](references/features.md) when explaining formulas, parameters, schemas or limitations.

The original extractor's foreground mask is the baseline. The approved full-pilot runner enables the frozen optional gradient correction; `correct_mask` itself defaults to disabled. Do not substitute earlier `row_background` or `local_median` candidates for the selected method. The correction changes foreground coverage and derived boxes/layout, not palettes or Canny density.

Do not introduce screenshot-specific crops, hardcoded application colors or automatic removal of genuine navigation. The approved run uses complete processed images, including inherited footer/frame artifacts. When artifact treatment is uncertain, document/manual-review it rather than silently cropping.

## Validate and report

Match screen IDs across metadata, scalars, palettes, boxes and layout records; check finite values, bounds, successful status and expected counts. For reproduction, compare saved numerical artifacts and configuration, verify deterministic behavior in the existing environment and confirm that prior inputs/outputs remain unchanged. For source changes, run checks that cover the changed behavior; for a documentation/copy-only task, verify links and byte/hash identity instead of rerunning image extraction.

Inspect saved originals, masks, edge maps and geometry/layout diagnostics relevant to the task, especially any activated correction and representative extremes. Separate numerical correctness from visual extraction quality. State which figures were actually inspected; generation alone is not inspection.

Deliver the requested results in a fresh versioned location with configuration/provenance, numerical differences when applicable, relevant visual evidence, a concise validation report and reproduction instructions. Define mathematical symbols in methodological explanations. State remaining limitations without inventing ground-truth accuracy. Components are not semantic UI elements, and extraction quality does not establish visual consistency.

Pairwise distances, M1 edge similarity, color tau and composite weights are downstream scoring choices. Do not modify extraction to implement them, or perform graph/classification work unless separately requested.
