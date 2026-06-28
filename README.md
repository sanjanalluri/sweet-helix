# sweet-helix | Glycan Point Cloud Metric Analysis

## Overview

This project investigates how four distance metrics, Chamfer Distance, Hausdorff Distance, Wasserstein Distance, and Silhouette Score, detect structural differences in 3D biological point clouds derived from prostate cancer glycan data. The analysis uses DBSCAN clustering, jitter-based data augmentation, and controlled parameter sweeps (translation and rotation) to systematically evaluate each metric's sensitivity to different types of geometric change.

---

## Research Question

**How do Silhouette Score, Chamfer Distance, Wasserstein Distance, and Hausdorff Distance detect differences in various types of structured 3D point clouds?**

This question is motivated by the need to compare glycan distributions across cancer samples from multiple datasets. As new cancer types are added to the study, a reliable understanding of what each metric captures and misses is essential for choosing the right comparison tools.

---

## Data

The dataset is a 3D PCA-reduced representation of prostate cancer glycan samples from three published sources:

- **Conroy** — prostate cancer glycan profiles
- **Nyalwidhe** — prostate cancer glycan profiles
- **Tajiri** — prostate cancer glycan profiles

Each sample is represented as a point in 3D space using its first three principal components (PC1, PC2, PC3), resulting in a single point cloud of **153 points**.

### File

```
prostate_data/Prostate_Glycans_pca_median_standardized_first3PC.csv
```

Columns: `Category`, `Sample_Name`, `PC1`, `PC2`, `PC3`

---

## Methods

### 1. Data Augmentation
To generate synthetic data, the real 153-point cloud is jitter-augmented by adding small Gaussian noise (scale = 0.5) to each point, producing 5 independent synthetic copies. Each copy is treated as a separate point cloud for downstream analysis.

### 2. DBSCAN Clustering
DBSCAN (`eps=10`, `min_samples=3`) is applied to the real data and each synthetic copy to identify clusters without requiring a human-specified cluster count. The algorithm consistently identifies 5 clusters and 3 outliers across all copies, confirming structural stability.

| Cluster | Points | Description |
|---|---|---|
| 0 | 117 | Large dominant cluster |
| 1 | 10 | Satellite cluster |
| 2 | 4 | Small isolated cluster |
| 3 | 9 | Satellite cluster |
| 4 | 9 | Satellite cluster |
| Outliers | 3 | Unclustered points |

### 3. Parameter Sweep
Two clusters are selected for comparison — Cluster 0 (A, n=117) and Cluster 1 (B, n=10). Two geometric gradients are applied to B in discrete steps:

- **Translation:** B is moved in the +PC1 direction in 5 steps of 5 units each
- **Rotation:** B is rotated around the PC3 axis in 5 steps of 45 degrees each

At each step, all four metrics are computed between A and the transformed B. This sweep is run on the real data and all 5 synthetic copies.

---

## Results

### Baseline Metrics (no transformation)
| Metric | Value |
|---|---|
| Chamfer | 29.85 |
| Hausdorff | 22.44 |
| Wasserstein | 9.32 |
| Silhouette | 0.86 |

### Metric Behavior Summary

| Metric | Translation | Rotation |
|---|---|---|
| Chamfer | Sensitive | Almost blind |
| Hausdorff | Sensitive | Slightly sensitive |
| Wasserstein | Sensitive | Completely blind |
| Silhouette | Sensitive | Completely blind |

### Key Findings

- All four metrics detected translational shifts, producing a U-shaped curve as B moved toward then past A
- Wasserstein was completely flat across all rotation steps — it is blind to rotation because it computes marginal distributions per axis independently
- Silhouette was also nearly flat across rotation steps, showing almost no sensitivity to orientation change
- Hausdorff was the only metric with meaningful sensitivity to rotation, making it the most geometrically complete of the four
- Chamfer showed minor fluctuation under rotation but was largely insensitive
- All findings were consistent and reproducible across 5 synthetic copies, validating that results are not artifacts of a single point cloud

### Conclusion

No single metric fully captures all types of structural differences between 3D point clouds. For datasets where cluster orientation may carry biological meaning such as glycan distributions across cancer types, using all four metrics together is necessary for a complete picture. Hausdorff should be prioritized when orientation sensitivity matters. Wasserstein and Silhouette are reliable for positional shifts but should not be used alone when rotational differences are expected.

---

## Repository Structure

```
.
├── prostate_data/
│   └── Prostate_Glycans_pca_median_standardized_first3PC.csv
├── analysis.ipynb
└── README.md
```

---

## Dependencies

Install all dependencies with:

```bash
pip install pandas numpy plotly scikit-learn scipy
```

| Package | Purpose |
|---|---|
| pandas | Data loading and manipulation |
| numpy | Numerical computation |
| plotly | 3D point cloud visualization |
| scikit-learn | DBSCAN clustering, Silhouette Score |
| scipy | Chamfer, Hausdorff, Wasserstein distance computation |

---

## How to Run

1. Clone the repository
2. Place the data file in a folder called `prostate_data/` in the same directory as the notebook
3. Install dependencies:
```bash
pip install pandas numpy plotly scikit-learn scipy
```
4. Open `analysis.ipynb` in Jupyter Lab:
```bash
jupyter lab
```
5. Run all cells in order from top to bottom

---

## Notes

- The notebook uses `os.getcwd()` to build file paths automatically, so it will work on any machine without path edits
- Plotly visualizations render inline in Jupyter Lab; if using classic Jupyter Notebook, set `pio.renderers.default = 'notebook'`
- DBSCAN parameters (`eps=10`, `min_samples=3`) were tuned for this specific dataset and coordinate scale (PC values range from approximately -38 to +7)
- Jitter scale for augmentation (`0.5`) was chosen to preserve cluster structure while introducing meaningful variation

## Sources
- Conroy — [citation pending]
- Nyalwidhe — [citation pending]
- Tajiri — [citation pending]
