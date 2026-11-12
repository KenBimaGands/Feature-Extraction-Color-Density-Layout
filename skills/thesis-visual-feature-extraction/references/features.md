# Feature definitions, correction and exported records

These describe the approved implementation. Read current source/configuration if they disagree with this reference; report the discrepancy before claiming a reproduction.

## Preprocessing and feature formulas

I is the processed RGB image; W and H are its width and height, N=WH its pixel count, and p a pixel position. Original images are EXIF-oriented, alpha-composited over white and resized with Lanczos to fit within 512×512 without upscaling, stretching or padding. The approved correction uses the complete processed image; no footer or synthetic-frame crop is applied.

**Color:** Let uj be unique RGB vector j, nj its pixel multiplicity, U the unique-color count, K=min(5,U), μk cluster centroid k, and aj the cluster assigned to uj. Weighted K-Means minimizes J=Σj nj||uj−μaj||², where J is squared RGB error and ||·|| is Euclidean norm. Seed=42, K-Means++ initialization, n_init=10 and Lloyd iterations are fixed; clustering is single-threaded. Cluster proportion qk=(Σj:aj=k nj)/N. Centroids are converted from float RGB to standard CIELAB using OpenCV. L* is lightness and a*,b* are chromatic axes; these are not encoded 8-bit Lab values. No mask is used to reweight palettes.

**Baseline foreground:** Perimeter RGB values are quantized in bins of width 16; the most frequent bin wins with lexicographic tie-breaking. B is its per-channel median RGB. M(p)=1 when ||I(p)−B||>25 and 0 otherwise. Foreground coverage F=(Σp M(p))/N. Σ denotes summation over the indicated pixels or unique-color indices. M is a binary heuristic mask, not semantic segmentation.

**Edges:** Convert independently to grayscale, Gaussian-blur with 5×5 window and sigma=1, then use OpenCV Canny thresholds 50/150. E(p) is 1 when the detector marks an edge, otherwise 0; edge density e=(Σp E(p))/N. Sigma is the blur standard deviation in pixels. Foreground correction does not change E or e.

**Boxes and layout:** Use eight-connected foreground components and retain components of at least 12 pixels for boxes. Small components still contribute to F. Boxes are normalized by processed width/height. For grid cell k, uk=area(Gk∩R)/area(Gk), where Gk is that cell's pixel region, R is the union of retained bounding-box regions, ∩ means intersection and area counts covered pixels. The grid has four columns and eight rows; k=0,…,31 follows row-major order. Overlapping boxes contribute once through the union. Components and rectangles are not semantic widgets.

## Frozen optional correction

`correct_mask(rgb, baseline, enabled=False)` copies the input mask and returns the revised mask plus diagnostic reasons. When enabled, all of these gates apply:

- Median row profiles in left and right side bands, each 3.5% of width, must each contain a long gradient run. Adjacent RGB steps must be at most 6, length at least 25% of image height, and RGB span at least 40.
- Baseline foreground coverage must exceed .5.
- Pixels whose maximum four-neighbor RGB step is at most 6 define smooth regions, joined with four-connectivity. A removable candidate must touch both actual image side borders and occupy at least .35 of the image area.

RGB span is the Euclidean norm of the three channel-wise ranges over the run. Choose the largest eligible region; ties choose the lowest component label. Suppress exactly that region from the baseline foreground; add no foreground pixels. Correction connectivity is four; final feature components remain eight-connected.

If R(p) indicates the selected region, M′(p)=M(p)(1−R(p)); otherwise M′=M. F′ is the sum of M′ divided by N. Recalculate boxes and layout from M′ using the existing geometry definition, preserving palettes and Canny density. The approved full runner enables this correction explicitly, while the baseline extractor remains uncorrected.

Large filled foreground regions can still be confused with smooth backgrounds; missed pale controls, fragmented text, frame/footer edges and coarse occupancy remain limitations. Do not infer accuracy percentages from mask-area changes or claim agreement with humans without annotated ground truth.

## Export schemas

| File | Meaning |
|---|---|
| `visual_features.csv` | `screen_id,status,error,original_width,original_height,width,height,foreground_coverage,edge_density,component_count`; corrected full run also has `correction_activated` |
| `color_palettes.json` | Screen-ID mapping to `colors` entries containing `rgb`, `lab`, `proportion`, plus `background_rgb` |
| `bounding_boxes.json` | Screen-ID mapping to normalized rectangle lists |
| `layout_vectors.csv` | `screen_id` plus `cell_00` through `cell_31` |
| `configuration.json` | Baseline settings, frozen correction parameters, activation setting and region description |
| `triggers.json`, `differences.csv` | Per-screen correction evidence and changes relative to the baseline |
| `run_summary.json` | Requested/successful counts, errors, activations and recorded checks |
| `source_sha256.json`, `protected_sha256.json` | Source provenance and protection evidence from the original run |
| `visualizations/`, `masks/` | Saved seven-panel comparisons and corrected masks; original six-panel baseline plots also contain Canny maps |

Validate unique screen-ID sets across records; fractions and occupancies must be finite in [0,1], palette proportions sum to one, and retained records have successful status. Preserve raw precision in CSV/JSON; rounded plot labels are display values.

## Downstream boundary

`pairwise_baseline.py` consumes saved descriptors. M1 edge similarity, exponential color normalization and equal-weight composite scoring are implemented downstream in `composite_baseline.py`, with outputs in `results/composite_baseline_v1/`. They do not alter extraction settings. Feature extraction is screenshot-local and navigation-independent; visual similarity is not a consistency classification.
