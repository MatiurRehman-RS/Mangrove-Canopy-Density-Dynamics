# Spatiotemporal Mangrove Canopy Density Dynamics — Indus Delta, Pakistan

Multi-temporal canopy density classification framework for the world's
largest arid-climate mangrove system using Sentinel-2 MSI imagery,
ICA-based spectral feature extraction, Spectral Angle Mapper
classification, stable-segment cross-temporal normalization, and
time-consistent OBIA in eCognition Developer.

> Manuscript under peer review — 2025

---

## Scientific Problem

Binary extent mapping — the standard output of operational mangrove
monitoring products including GMW v3.0 — cannot distinguish pioneer-
stage sparse vegetation from structurally mature closed-canopy forest.
This limitation is particularly consequential in the Indus Delta, where
areal expansion has been widely documented but the structural quality
of recovering stands remained entirely uncharacterized.

Three specific methodological conditions were required to resolve
density-stratified canopy structure in this environment, none of which
had been jointly satisfied by prior work:

1. Isolation of canopy density variation from spectrally mixed intertidal
   signals where pixels integrate canopy, open water, and exposed
   substrate in variable proportions
2. Cross-temporal spectral comparability independent of atmospheric
   correction uncertainty and tidal-state variability between acquisition
   dates
3. Spatially consistent object boundaries across all epochs so that
   detected transitions reflect genuine ecological change rather than
   segmentation instability

---

## Study Area

**Indus Delta**, Sindh Province, Pakistan  
Eight spatially discrete mangrove cluster units, central and western delta  
Total consistently delineated area: **36,650 ha**  
Climate: Hot arid, mean annual precipitation 100–200 mm  
Dominant species: *Avicennia marina* (>90% canopy cover)  
Tidal regime: Mesotidal, mean spring range 2–3 m  
Conservation context: Sindh Forest Department jurisdiction;
Delta Blue Carbon (DBC) project operational boundary

---

## Datasets

| Sensor | Product | Acquisition Date | Bands | Resolution |
|---|---|---|---|---|
| Sentinel-2A MSI | L2A Surface Reflectance | 15 January 2016 | B2, B3, B4, B8, B11, B12 | 10m / 20m |
| Sentinel-2A MSI | L2A Surface Reflectance | 14 January 2020 | B2, B3, B4, B8, B11, B12 | 10m / 20m |
| Sentinel-2A MSI | L2A Surface Reflectance | 17 January 2025 | B2, B3, B4, B8, B11, B12 | 10m / 20m |

January acquisitions selected for dry-season low-tide conditions —
maximum canopy spectral contrast with surrounding intertidal substrate.
SWIR bands (B11, B12) bilinearly resampled from 20m to 10m.
Source: Copernicus Data Space Ecosystem.

---

## Methodology

Five-stage integrated processing pipeline:

```
Sentinel-2 L2A (3 epochs)
        │
        ▼
[Stage 1] Independent Component Analysis (ICA)
          FastICA — deflationary scheme, hyperbolic tangent nonlinearity
          4 components retained via kurtosis stability criterion
          (kurtosis excess >1.5 across 50 runs)
        │
        ▼
[Stage 2] Spectral Angle Mapper (SAM) Classification
          Endmembers: 8 ROIs, spectrally homogeneous dense canopy
          Consistently sampled across identical geographic locations
          per epoch — fixed spectral library across all dates
          Output: per-pixel angular distance raster per epoch
        │
        ▼
[Stage 3] Time-Consistent OBIA Segmentation (eCognition Developer 10.8)
          Joint segmentation of 3-epoch SAM stack as multi-layer input
          Level 1 MRS: scale=45, shape=0.12, compactness=0.7
          Level 2 MRS: scale=15, shape=0.3, compactness=0.5
          Level 3: Chessboard 10×10m tessellation
          Output: fixed spatial object framework — same boundaries
          applied to all three epochs
        │
        ▼
[Stage 4] Stable-Segment Cross-Temporal Normalization
          OLS regression on progressively restricted SAM subsets
          (100% to 10%, 10% increments)
          Stability threshold: 20th percentile
          (R² ≈ 0.95–0.96, RMSE ≈ 0.061–0.069 for both epoch pairs)
          Min-max normalization anchored to stable-ensemble bounds
          Output: SAM rasters normalized to common 0–1 range
        │
        ▼
[Stage 5] Multi-Threshold Otsu Density Classification
          Binary Otsu threshold (t=0.3027): mangrove / non-mangrove
          Multi-threshold Otsu on pooled 3-epoch per-object distribution:
          t₁=0.15, t₂=0.26, t₃=0.37
          Single unified threshold set — consistent class boundaries
          across all epochs
```

---

## Density Classes

| Class | SAM Range | Ecological Interpretation |
|---|---|---|
| Dense Mangrove | 0.00 – 0.15 | Closed-canopy, structurally mature stands |
| Medium Mangrove | 0.15 – 0.26 | Moderate closure, mixed-age, partial gap fraction |
| Sparse Mangrove | 0.26 – 0.37 | Open canopy, pioneer fringe, post-disturbance regeneration |
| Non-Mangrove | > 0.37 | Bare substrate, open water, upland |

---

## Results

### Canopy Density Area by Epoch

| Density Class | 2016 (ha) | 2020 (ha) | 2025 (ha) | Change 2016–2025 (ha) |
|---|---|---|---|---|
| Dense Mangrove | 7,466 | 10,127 | 12,542 | +5,075 |
| Medium Mangrove | 7,697 | 10,524 | 13,048 | +5,351 |
| Sparse Mangrove | 10,454 | 11,094 | 9,484 | -970 |
| Non-Mangrove | 11,033 | 4,905 | 1,576 | -9,456 |

Dense + Medium combined: **41.3%** of extent (2016) → **69.8%** (2025)  
Non-mangrove contraction: **85.7%** reduction from 2016 baseline

### Full-Period Transition Matrix (2016–2025)

| Source 2016 → Target 2025 | Dense (ha) | Medium (ha) | Sparse (ha) | Non-MG (ha) | Persistence |
|---|---|---|---|---|---|
| Dense Mangrove | **6,631** | 703 | 77 | 54 | 88.8% |
| Medium Mangrove | 2,996 | **3,637** | 980 | 82 | 47.3% |
| Sparse Mangrove | 1,543 | 4,935 | **3,492** | 483 | 33.4% |
| Non-Mangrove | 1,370 | 3,772 | 4,934 | **956** | 8.7% |

**Upward transitions: 19,552 ha**  
**Downward transitions: 2,382 ha**  
Upward:downward ratio = **8.2:1**

### Two-Phase Recovery

**Phase 1 (2016–2020) — Pioneer Colonization:**
Non-mangrove → sparse mangrove dominant (48.7% of 2016 non-vegetated area)

**Phase 2 (2020–2025) — Canopy Consolidation:**
Sparse → medium and medium → dense transitions dominant
Dense mangrove persistence increased to 91.1%

### Classification Accuracy

| Epoch | OA (%) | Kappa |
|---|---|---|
| 2016 | 78.1 | 0.708 |
| 2020 | 79.7 | 0.729 |
| 2025 | 82.5 | 0.767 |

320 stratified random validation samples per epoch (80 per class).
Reference labels from Google Earth Pro visual interpretation
(±4 month temporal offset).

---

## Key Finding

The Indus Delta mangrove ecosystem underwent genuine structural
densification — not merely areal expansion — under active restoration
between 2016 and 2025. This finding directly addresses the apparent
contradiction between documented areal growth and declining NDVI
trends (Raza et al., 2024): newly colonized sparse pioneer stands
inherently produce low vegetation index values, which recover as
canopy closure and biomass accumulate through succession.

---

## Repository Structure

```
├── src/
│   ├── ica_sam_pipeline.py          ← ICA + SAM processing
│   ├── normalization.py             ← stable-segment normalization
│   ├── otsu_classification.py       ← multi-threshold Otsu
│   ├── transition_matrix.py         ← change analysis
│   └── accuracy_assessment.py      ← confusion matrix, OA, Kappa
├── data/
│   └── README.md                    ← data sources, no raw imagery
├── results/
│   ├── density_area_by_epoch.csv
│   ├── transition_matrix_full.csv
│   ├── transition_matrix_2016_2020.csv
│   ├── transition_matrix_2020_2025.csv
│   └── accuracy_results.csv
├── figures/
│   ├── density_maps_2016_2020_2025.png
│   ├── transition_chord_diagram.png
│   ├── spatial_transition_maps.png
│   ├── otsu_histogram.png
│   └── normalization_sensitivity.png
├── docs/
│   └── pipeline_methodology.md
├── requirements.txt
├── LICENSE
└── README.md
```

---

## Data Sources

- Sentinel-2A MSI L2A: Copernicus Data Space Ecosystem
  (https://browser.dataspace.copernicus.eu)
- Study area boundary: Custom digitized vector
- Validation reference: Google Earth Pro high-resolution imagery
- No raw satellite imagery stored — file sizes exceed repository limits

---

## Dependencies

```
Python 3.10+
numpy, scipy, scikit-learn, rasterio, geopandas
matplotlib, seaborn
scikit-image (multi-threshold Otsu)
```
eCognition Developer 10.8 (Trimble) — required for OBIA segmentation.
Segmentation performed interactively; ruleset description in docs/.

---

## Publication

*Spatiotemporal Analysis of Mangrove Canopy Density Dynamics
in the Indus Delta, Pakistan.*  
Manuscript under peer review — 2025.

**First author:** Mati ur Rehman  
Centre for Geographic Information Science (CGIS),
University of Punjab, Lahore, Pakistan
