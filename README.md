# Pilot visual feature extraction

Extracts color palettes, two independent density measurements, and spatial layout features for the thesis *Graph-Based Mobile UI Visual Consistency Evaluation*. No similarity, graph, classification, or combined score is computed. Original inputs are read only.

## Reproduce

This repository contains the baseline feature extraction code only. Supply your dataset locally at the metadata paths below; screenshots, metadata, generated results, notebooks, and experimental corrections are not included.

Create a Python environment and install the versions in `requirements-lock.txt` if you do not already have the project environment. The lock captures the original full environment, including notebook packages; extraction uses the libraries listed below.

Run from this directory using the existing environment:

```sh
.venv/bin/python -m unittest -v test_features
.venv/bin/python extract_features.py --output results/smoke --screen-ids APP01-FLOW01-S00 APP01-FLOW01-S02 APP01-FLOW02-S04 --plots
.venv/bin/python extract_features.py --plots
```

The smoke selection covers a splash, registration form, and portfolio dashboard. Inspect its three plots before interpreting the full run. 

Libraries: Pillow (decoding, orientation, resize), NumPy (arrays), scikit-learn (K-Means), OpenCV (CIELAB, grayscale, Gaussian blur, Canny, connected components), threadpoolctl (single-thread clustering), Matplotlib (plots). Tests use unittest. `requirements-lock.txt` captures the existing environment; the dependency snapshot is included for reproduction. Fixed seeds and single-thread clustering support reproducibility within this environment; cross-platform numerical identity is not promised.

## Inputs and workflow

Actual metadata is `PILOT/screenshots/screens.csv`, with UTF-8 BOM, original `screen_id`, and relative `image_path`. Paths resolve relative to that CSV and must stay inside its directory. Duplicate IDs stop the run. `flows.csv` and `transitions.csv` are retained but unused: extraction is per screenshot.

1. Decode PNG/JPEG/WEBP/BMP/TIFF and reject missing, corrupt, unsupported, or excessively large images using Pillow checks. Apply EXIF orientation, composite transparency over white, convert to RGB.
2. Resize with Lanczos to fit inside 512 × 512, preserving aspect ratio, without upscaling or padding. Record original oriented and processed dimensions. Images smaller than the grid are rejected.
3. Cluster all processed RGB pixels via their unique values and multiplicities. No pixel sampling is used.
4. Independently derive foreground mask and blurred grayscale Canny edges.
5. Extract connected foreground components, filter small components, and calculate occupancy of the union of retained rectangles.
6. Write numerical results, configuration, and optional six-panel diagnostic plots.

All feature areas refer to the processed image. Resizing can remove small details and introduce new colors; fixed preprocessing is necessary when interpreting these descriptors.

## Mathematics and notation

Let the processed RGB image be I, width W pixels, height H pixels, total pixel count N = WH, and pixel position p = (x,y), with x in {0,…,W−1} and y in {0,…,H−1}. I(p) is its three-channel RGB vector, with each channel between 0 and 255. Areas below are pixel counts. Ratios are dimensionless.

### Weighted color palette: established K-Means and CIELAB

Let U be the number of unique RGB vectors, u_j the j-th unique vector, and n_j its pixel count, for j = 1,…,U. Set K = min(5,U). Weighted K-Means minimizes

    J = Σ_j n_j ||u_j − μ_(a_j)||²₂

Here J is squared-error cost, μ_k is RGB centroid of cluster k, a_j is the cluster assigned to u_j, k ranges from 1 to K, and ||·||₂ is Euclidean norm. Unique values with sample weights give the full-pixel objective. K-Means++ initialization, seed 42, ten initializations, and Lloyd iterations are used. K-Means converges to a local solution; it does not guarantee a global optimum.

    q_k = (Σ_{j:a_j=k} n_j) / N;    Σ_k q_k = 1

q_k is cluster k's normalized pixel proportion. Each μ_k is converted, without integer rounding, from sRGB to standard D65 CIELAB with OpenCV float RGB inputs scaled to [0,1]. In standard notation L* = 116f(Y/Yn)−16, a* = 500[f(X/Xn)−f(Y/Yn)], b* = 200[f(Y/Yn)−f(Z/Zn)]. X,Y,Z are tristimulus coordinates after sRGB linearization; Xn,Yn,Zn are D65 reference-white tristimulus values. f(t) = t^(1/3) when t > δ³, otherwise t/(3δ²)+4/29, where t is a normalized tristimulus value and δ = 6/29. L* represents lightness (nominal 0–100); a* and b* are signed chromatic axes. These are standard coordinates, not OpenCV's encoded 8-bit Lab. OpenCV uses numerical approximations. RGB centroids are means in encoded RGB, as requested, rather than means in Lab. Palettes are ordered by decreasing q_k, with RGB tie breaks.

Example: 75 white pixels and 25 black pixels yield K=2, proportions .75 and .25, approximately Lab [100,0,0] and [0,0,0].

### Foreground coverage: implementation heuristic

Collect perimeter RGB pixels (corners once). Quantize each channel into bins of width 16 using floor division. Choose the most frequent RGB bin; ties choose the lexicographically first bin. Let B be the per-channel median of original perimeter pixels in that bin. Define

    M(p) = 1 if ||I(p) − B||₂ > τ, otherwise 0
    F = Σ_p M(p) / N

B is the estimated background RGB vector, τ is the RGB-distance threshold (25), M is the binary foreground mask, and F is foreground coverage in [0,1]. This mask is not blurred or cleaned for coverage. If 800 of 3,200 pixels are foreground, F=.25. This background estimator is a heuristic, not a semantic foreground segmentation method.

### Canny density: established detector, configured thresholds

Convert RGB to OpenCV grayscale, blur with a 5 × 5 Gaussian kernel of standard deviation σ=1 pixel, then use Canny thresholds 50 and 150 (3 × 3 Sobel aperture, default L1 gradient, default OpenCV border behavior).

    D = Σ_p E(p) / N

E(p) equals 1 when Canny marks an edge pixel, otherwise 0; D is edge density in [0,1]. For 160 marked pixels among 3,200, D=.05. F and D remain separate; neither measures visual consistency by itself.

### Layout: established components, heuristic filtering and grid

Use 8-connected components of M (foreground pixels joined horizontally, vertically, or diagonally). Keep components with area at least A_min=12 pixels. This absolute cutoff is a noise heuristic at the configured working resolution, not a validated UI-element detector.

For each retained component, let x0,y0 be its inclusive top-left position and x1,y1 its exclusive bottom-right position. Output [x0/W, y0/H, x1/W, y1/H]. These bounds lie in [0,1]. Let R be the union of pixels inside all retained bounding rectangles, including empty space inside a rectangle. Rectangles are rasterized into a single binary union mask to prevent overlap double counting.

Define column boundaries c_i=floor(iW/4), i=0,…,4, and row boundaries r_j=floor(jH/8), j=0,…,8. Cell G_(j,i) consists of pixels with c_i ≤ x < c_(i+1) and r_j ≤ y < r_(j+1). Its occupancy is

    v_(4j+i) = |R ∩ G_(j,i)| / |G_(j,i)|

Here i=0,…,3 indexes columns, j=0,…,7 indexes rows, |·| means pixel count, ∩ means intersection, and v is the 32-value vector in row-major order (left to right, top to bottom). Every v entry is in [0,1]; their sum need not be 1. A cell with union coverage of 60 of its 100 pixels has occupancy .6. Non-divisible image dimensions produce slightly different cell sizes. The 4 × 8 grid is a chosen spatial descriptor, not a mathematically optimal resolution.

## Outputs and configuration

- `results/visual_features.csv`: every selected original screen ID, success/error status, error detail, dimensions, F, D, retained component count. Failed rows have blank feature values.
- `results/color_palettes.json`: successful IDs mapped to weighted RGB/Lab colors and estimated background RGB.
- `results/bounding_boxes.json`: successful IDs mapped to normalized exclusive-endpoint boxes.
- `results/layout_vectors.csv`: successful IDs and `cell_00` through `cell_31`.
- `results/config.json`: exact extraction parameters.
- `results/visualizations/0000.png`, etc.: metadata-order indices, including failed-row positions. Panels show processed screenshot, weighted palette, foreground mask, edges, bounding boxes, and occupancy grid overlay. Look up the corresponding ID by row index in `visual_features.csv`.

`--metadata`, `--output`, `--screen-ids`, and `--plots` select inputs and outputs. `--config settings.json` overrides dataclass defaults using JSON, e.g. `{"max_dimension": 512, "background_distance": 25.0}`. Other parameters are `seed`, `min_component_area`, `blur_kernel` (positive odd integer), `blur_sigma`, `canny_low`, `canny_high`. Keep the same configuration across screens. Running into an existing output directory overwrites numerical files; use a fresh output directory when comparing experiments. Old visualization files are not removed automatically.

## Validation and limitations

Tests decode every metadata image; check exact repeatability, K≤5, normalized proportions, density bounds, valid boxes, 32 bounded occupancy entries, white/black analytic examples, overlapping rectangle union, and missing/corrupt-image continuation. The CLI returns nonzero if any image fails while preserving successful results and error rows.

Background estimates can fail with gradients, photos, border decorations, or multiple background regions. Content similar to the estimated background is missed. Foreground components can merge adjacent controls, text, illustrations, and containers; their rectangles can cover large empty interiors. No OCR, semantic UI detection, status/navigation-bar removal, or morphology is applied. Canny is sensitive to resolution, blur, contrast, and chosen thresholds. Palette proportions describe resized pixels, not perceptual importance. These descriptors require visual inspection before later similarity work; no thresholds were calibrated here.

Observed in the three-screen smoke inspection: Mobbin attribution footers, device status bars, and home indicators contribute foreground/edges/layout. Pale input outlines may fall below the foreground threshold. A line chart becomes a rectangle containing considerable empty space. All are retained by the specified extraction method; crop rules or semantic segmentation would require a separate methodological decision.


