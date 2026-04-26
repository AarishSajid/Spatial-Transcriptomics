# Section 1 — Basic Scanpy Spatial Analysis

Standard scRNA-seq clustering pipeline applied to a 10x Genomics Visium human lymph node dataset, with spatial projection of results onto the H&E tissue image.

---

## Dataset

| Property | Value |
|----------|-------|
| Sample | Human Lymph Node (V1_Human_Lymph_Node) |
| Spots | 4,035 |
| Genes | 36,601 |
| Image | H&E included |

---

## Setup

```bash
pip install scanpy seaborn igraph leidenalg
```

---

## Workflow

### Step 1 — Initialize Environment

Configure plot styling and logging verbosity before loading any data.

```python
import scanpy as sc
import matplotlib.pyplot as plt

sc.set_figure_params(facecolor="white", figsize=(8, 8))
sc.settings.verbosity = 3  # Verbose logging for reproducibility
```

### Step 2 — Load Data and Compute QC Metrics

Load the Visium dataset directly via Scanpy's built-in loader. Flag mitochondrial genes and compute per-spot quality metrics.

```python
adata = sc.datasets.visium_sge(sample_id="V1_Human_Lymph_Node")
adata.var_names_make_unique()

# Flag mitochondrial genes
adata.var["mt"] = adata.var_names.str.startswith("MT-")
sc.pp.calculate_qc_metrics(adata, qc_vars=["mt"], inplace=True)
```

### Step 3 — Filter Low-Quality Spots and Genes

Remove spots with very low or suspiciously high transcript counts, spots with high mitochondrial content (a marker of cell death or damage), and genes detected in fewer than 10 spots.

```python
sc.pp.filter_cells(adata, min_counts=5000)
sc.pp.filter_cells(adata, max_counts=35000)
adata = adata[adata.obs["pct_counts_mt"] < 20].copy()
sc.pp.filter_genes(adata, min_cells=10)
```

### Step 4 — Normalize and Select Highly Variable Genes

Normalize total counts per spot, apply log1p transformation, and select the top 2,000 highly variable genes for downstream dimensionality reduction.

```python
sc.pp.normalize_total(adata, inplace=True)
sc.pp.log1p(adata)
sc.pp.highly_variable_genes(adata, flavor="seurat", n_top_genes=2000)
```

### Step 5 — Dimensionality Reduction and Clustering

Run PCA to compress the gene expression space, build a neighbor graph, compute a UMAP embedding for visualization, and assign spots to Leiden clusters.

```python
sc.pp.pca(adata)
sc.pp.neighbors(adata)
sc.tl.umap(adata)
sc.tl.leiden(adata, key_added="clusters", flavor="igraph")
```

### Step 6 — Spatial Projection

Overlay the computed cluster labels onto the H&E tissue image to reveal the spatial organization of each transcriptional cluster.

```python
# Full tissue view
sc.pl.spatial(adata, img_key="hires", color="clusters", size=1.5)

# Zoomed view of specific clusters
sc.pl.spatial(
    adata, img_key="hires", color="clusters",
    groups=["5", "9"], crop_coord=[7000, 10000, 0, 6000]
)
```

### Step 7 — Differential Expression and Marker Genes

Identify marker genes for each cluster using a t-test and visualize selected markers spatially to confirm biological relevance.

```python
sc.tl.rank_genes_groups(adata, "clusters", method="t-test")

# CR2 is a known follicular B-cell marker — confirms cluster 9 identity
sc.pl.spatial(adata, img_key="hires", color=["clusters", "CR2"])
```

---

## Results

| File | Description |
|------|-------------|
| `01_qc_metrics.jpeg` | QC metric distributions — total counts, gene counts, mitochondrial percentage |
| `02_umap_clusters.jpeg` | UMAP embedding colored by cluster identity |
| `03_spatial_counts.jpeg` | Spatial map of total transcript counts per spot |
| `04_spatial_clusters.jpeg` | Leiden clusters projected onto the H&E tissue section |
| `05_tissue_morphology.jpeg` | Zoomed spatial view of follicular cluster 9 |
| `06_marker_genes_heatmap.jpeg` | Top marker genes per cluster |
