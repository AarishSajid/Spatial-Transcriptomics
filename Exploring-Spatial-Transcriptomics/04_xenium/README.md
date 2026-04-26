# Section 4 — Xenium Single-Cell Spatial Analysis

Subcellular transcript mapping and spatial graph analysis of human lung cancer tissue using the 10x Genomics Xenium platform. Unlike Visium, Xenium resolves individual cells rather than spots.

---

## Dataset

Human lung cancer Xenium replicate 1. Data must be downloaded from 10x Genomics and converted to Zarr format before running this workflow.

---

## Setup

```python
import spatialdata as sd
import squidpy as sq
import scanpy as sc
```

---

## Workflow

### Step 1 — Load Xenium Data

Read the Xenium output directory as a SpatialData object backed by Zarr. This format efficiently handles millions of individual transcript coordinates without loading everything into memory.

```python
sdata = sd.read_zarr("Xenium.zarr")
adata = sdata.tables["table"]
```

### Step 2 — Quality Control and Normalization

Filter out cells with very low transcript counts, normalize total counts per cell, and apply log1p transformation. Then run PCA and Leiden clustering.

```python
sc.pp.filter_cells(adata, min_counts=10)
sc.pp.normalize_total(adata)
sc.pp.log1p(adata)
sc.pp.pca(adata)
sc.pp.neighbors(adata)
sc.tl.umap(adata)
sc.tl.leiden(adata)
```

### Step 3 — Build Spatial Neighbor Graph via Delaunay Triangulation

Because Xenium cells are irregularly positioned (not on a regular grid like Visium spots), neighbors are determined by Delaunay triangulation — connecting cell centers into triangles to define a physically meaningful adjacency structure.

```python
sq.gr.spatial_neighbors(adata, coord_type="generic", delaunay=True)
```

### Step 4 — Centrality Analysis

Compute graph centrality scores for each cluster to identify which cell types serve as structural or communicative hubs within the tumor microenvironment.

```python
sq.gr.centrality_scores(adata, cluster_key="leiden")
sq.pl.centrality_scores(adata, cluster_key="leiden")
```

### Step 5 — Spatial Co-occurrence

Analyze how the spatial proximity between cell type pairs changes across distance bins to characterize tumor niche organization.

```python
sq.gr.co_occurrence(adata, cluster_key="leiden")
sq.pl.co_occurrence(adata, cluster_key="leiden")
```

---

## Results

| File | Description |
|------|-------------|
| `01_spatial_clusters.jpeg` | Over 11,000 individual cells colored by Leiden cluster at true single-cell resolution |
| `02_centrality_scores.jpeg` | Centrality metrics per cluster — identifies hub cell types in the tumor microenvironment |
| `03_co_occurrence.jpeg` | Co-occurrence curves revealing tight tumor cell aggregation patterns distinct from healthy stroma |
