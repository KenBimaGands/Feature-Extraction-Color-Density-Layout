# Feature extraction — approved thesis snapshot

Updated 13 November 2026. The current approved extraction is the original baseline plus the frozen optional foreground correction, evaluated first on 12 screens and then applied to all 82 pilot screenshots. This folder now contains its source and saved full-pilot outputs. No extraction was rerun during this update.

| Location | Purpose |
|---|---|
| [approved_feature_extraction_v1/](approved_feature_extraction_v1/) | Current approved source snapshot, configuration and all 82 saved results |
| [feature_extraction_github/](feature_extraction_github/) | Original baseline code package, preserved unchanged |
| [run_01/](run_01/) | Original 12-screen correction experiment, preserved unchanged |
| [baseline_methodology.md](baseline_methodology.md) | Previous README, retained as the baseline's mathematical/learning reference |

## Current results

The approved full run is [results_corrected_pilot](approved_feature_extraction_v1/results_corrected_pilot/):

- [visual_features.csv](approved_feature_extraction_v1/results_corrected_pilot/visual_features.csv): 82 successful records with foreground coverage, independent Canny edge density and component count.
- [color_palettes.json](approved_feature_extraction_v1/results_corrected_pilot/color_palettes.json): weighted palettes, maximum K=5, RGB and CIELAB centroids.
- [bounding_boxes.json](approved_feature_extraction_v1/results_corrected_pilot/bounding_boxes.json) and [layout_vectors.csv](approved_feature_extraction_v1/results_corrected_pilot/layout_vectors.csv): retained component rectangles and 4-column × 8-row union occupancy.
- [configuration.json](approved_feature_extraction_v1/results_corrected_pilot/configuration.json), [triggers.json](approved_feature_extraction_v1/results_corrected_pilot/triggers.json) and [differences.csv](approved_feature_extraction_v1/results_corrected_pilot/differences.csv): parameters, correction decisions and baseline differences.
- [visualizations/](approved_feature_extraction_v1/results_corrected_pilot/visualizations/) and [masks/](approved_feature_extraction_v1/results_corrected_pilot/masks/): saved diagnostic comparisons and masks.
- [run_summary.json](approved_feature_extraction_v1/results_corrected_pilot/run_summary.json): 82/82 successful; correction activated only for `APP02-FLOW01-S01`; the other 81 screens retain their baseline features.

The selected **method** comes from the 12-screen `run_01` experiment; the current **complete results** are the separate 82-screen approved run. These are different evaluation scopes.

## What changed in extraction

The original `extract_features.py` remains the baseline. The frozen `correct_mask` function optionally suppresses one large, smooth region connected across both image sides when both side profiles show sufficient gradient evidence and the baseline mask has high coverage. The approved full-pilot runner explicitly enables this correction. The correction function defaults to disabled for other callers.

Correction updates foreground coverage and the connected-component boxes/layout derived from the corrected mask. Palette extraction, K-Means/CIELAB definitions and Canny edge density are unchanged. Full processed screenshots are retained: no new cropping, footer removal or device-frame policy was introduced by this selected correction. Acquisition artifacts, pale boundaries and semantic interpretation remain limitations. Connected components are not semantic UI elements.

See the [snapshot guide](approved_feature_extraction_v1/README.md) for source locations, parameters, verification and reproduction. The baseline's formulas remain in [baseline_methodology.md](baseline_methodology.md); correction explanations are preserved in the [original correction documentation](approved_feature_extraction_v1/source/experiments/final_baseline_correction/README.md).

## Later analysis stages

Extraction records remain separate from scoring. The main project additionally contains:

- [Traditional pairwise methods](../PAIRWISE_METHODS.md) and [saved pairwise results](../results_pairwise_baseline/).
- [Edge formulation evaluation](../experiments/edge_similarity_evaluation/edge_similarity_evaluation_report.md): M1 subsequently selected provisionally for composite scoring; this does not change extracted edge densities.
- [Composite baseline v1](../results/composite_baseline_v1/README.md): M1 edge, color tau=10 and equal Color/Density/Layout weighting, with sensitivity analysis.

These links refer to the surrounding thesis workspace; downstream scoring has not been folded into the extraction algorithm or copied into this snapshot. This update is local; nothing was committed or uploaded to GitHub.

## Reusable Codex skill

The [thesis-visual-feature-extraction skill](skills/thesis-visual-feature-extraction/SKILL.md) captures the approved extraction workflow, feature formulas, output schemas and preservation rules. Its supporting references explain reproduction and visual review. Invoke it as `$thesis-visual-feature-extraction` once available in Codex's skill catalog. The skill uses the existing project/environment; it does not bundle or rerun the dataset.
