# Section 3 — Visium H&E Spatial Graph Analysis

Statistical modeling of spatial tissue organization using neighborhood enrichment, co-occurrence curves, ligand-receptor screening, and Moran's I on a mouse brain H&E Visium section.

---

## Dataset

Mouse brain Visium data with H&E staining. Available via the Squidpy built-in dataset loader.

---

## Setup

```python
import scanpy as sc
import squidpy as sq
import numpy as np
```

---

## Workflow

### Step 1 — Build Spatial Neighbor Graph

Construct an adjacency matrix from spot coordinates. This graph defines which spots are physical neighbors and underpins all subsequent spatial statistics.

```python
sq.gr.spatial_neighbors(adata)
```

### Step 2 — Neighborhood Enrichment

Run a permutation test to determine which pairs of clusters are found adjacent to each other more (or less) often than expected by chance. Positive enrichment scores indicate clusters that are statistically likely to border each other.

```python
sq.gr.nhood_enrichment(adata, cluster_key="cluster")
sq.pl.nhood_enrichment(adata, cluster_key="cluster")
```

### Step 3 — Co-occurrence Analysis

Compute conditional cluster co-occurrence as a function of increasing spatial distance. This reveals how the probability of finding a specific cluster near another cluster changes with distance, identifying spatially structured relationships.

```python
sq.gr.co_occurrence(adata, cluster_key="cluster")
sq.pl.co_occurrence(adata, cluster_key="cluster", clusters="Hippocampus")
```

### Step 4 — Spatially Variable Gene Detection (Moran's I)

Identify genes whose expression is non-randomly distributed across the tissue. A high Moran's I score indicates that nearby spots tend to share similar expression levels for that gene.

```python
sq.gr.spatial_autocorr(adata, mode="moran", n_perms=100)
```

### Step 5 — Ligand–Receptor Analysis

Screen for ligand-receptor pairs that are co-expressed across neighboring clusters, identifying potential sites of intercellular signaling at anatomical boundaries.

```python
sq.gr.ligrec(adata, n_perms=100, cluster_key="cluster")
```

---

## Results

| File | Description |
|------|-------------|
| `01_spatial_clusters.jpeg` | H&E tissue section with annotated anatomical cluster labels |
| `02_neighborhood_enrichment.jpeg` | Enrichment heatmap — confirms Hippocampus and Pyramidal layer as statistically significant neighbors |
| `03_co_occurrence_hippocampus.jpeg` | Co-occurrence decay curve showing how hippocampal cluster proximity changes with distance |
