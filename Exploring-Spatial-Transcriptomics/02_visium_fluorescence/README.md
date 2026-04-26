# Section 2 — Visium Fluorescence Analysis

Image-based feature extraction and clustering using fluorescence channels (DAPI, NEUN, GFAP) on a mouse brain Visium dataset. Demonstrates that morphological information can resolve tissue structure beyond what gene expression alone captures.

---

## Dataset

Mouse brain Visium data with fluorescence channels. Available via the Squidpy built-in dataset loader.

---

## Setup

```python
import scanpy as sc
import squidpy as sq
import matplotlib.pyplot as plt
```

---

## Workflow

### Step 1 — Load Data

Load the pre-processed AnnData object and its accompanying fluorescence image from the Squidpy dataset collection.

```python
img = sq.datasets.visium_fluo_image_crop()
adata = sq.datasets.visium_fluo_adata_crop()
```

### Step 2 — Image Preprocessing and Nucleus Segmentation

Smooth the fluorescence image to reduce noise, then apply Watershed segmentation on the DAPI channel to delineate individual nuclei.

```python
sq.im.process(img=img, layer="image", method="smooth")
sq.im.segment(
    img=img,
    layer="image_smooth",
    method="watershed",
    channel=0  # DAPI channel
)
```

### Step 3 — Extract Image-Derived Features

For each Visium spot, compute segmentation-based features including estimated cell count, mean channel intensity (NEUN/GFAP), and texture statistics.

```python
sq.im.calculate_image_features(
    adata, img,
    features="segmentation",
    layer="image",
    key_added="features_segmentation"
)
```

### Step 4 — Cluster on Image Features

Build a separate AnnData object containing only the image features, then scale, run PCA, and apply Leiden clustering. This clustering uses no gene expression data whatsoever.

```python
sc.pp.scale(features_adata)
sc.pp.pca(features_adata)
sc.tl.leiden(features_adata)
```

---

## Results

| File | Description |
|------|-------------|
| `01_spatial_clusters.jpeg` | Gene-expression Leiden clusters overlaid on the tissue section |
| `02_segmentation_comparison.jpeg` | Side-by-side comparison of raw DAPI signal and computed nucleus masks |
| `03_image_features_vs_clusters.jpeg` | Image-feature clusters vs. gene-expression clusters — image features correctly subdivide the hippocampus into anatomical sub-layers |
