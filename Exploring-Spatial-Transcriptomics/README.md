# Exploring Spatial Transcriptomics

**Author:** Aarish Sajid | **Reg No:** 454318

An exploration of spatial biology using 10x Genomics Visium and Xenium platforms, combining transcriptomic analysis with spatial tissue context.

---

## Overview

Traditional transcriptomics — whether bulk or single-cell — discards one of the most important properties of a cell: its location. Cells do not act in isolation; their function is deeply tied to their physical environment and their neighbors. Spatial transcriptomics preserves this context by mapping gene expression directly onto tissue sections.

This project works across two scales of resolution:

- **Visium (55 µm spots)** — captures RNA from small groups of cells per spot, overlaid on stained tissue images
- **Xenium (subcellular)** — maps individual transcripts to single cells with precise spatial coordinates

The analysis pipeline uses **Scanpy** for standard single-cell clustering, **Squidpy** for spatial statistics and image features, and the **SpatialData/Zarr** ecosystem for handling high-resolution Xenium data.

---

## Repository Structure

```
Exploring-Spatial-Transcriptomics/
│
├── 01_basic_scanpy/              # Section 1 — Standard Visium clustering pipeline
│   ├── scanpyBasic.ipynb
│   ├── results/
│   └── README.md
│
├── 02_visium_fluorescence/          # Section 2 — Image-based feature analysis
│   ├── results/
│   └── README.md
│
├── 03_visium_hne/                   # Section 3 — Spatial graph statistics on H&E
│   ├── results/
│   └── README.md
│
├── 04_04_xenium/                   # Section 4 — Single-cell Xenium analysis
│   ├── results/
│   └── README.md
│
└── docs/                     # GitHub Pages visualization
    └── index.html
```

---

## Sections

| # | Section | Dataset | Focus |
|---|---------|---------|-------|
| 1 | [Basic Scanpy Spatial](01_basic_scanpy/) | Human Lymph Node (Visium) | QC, clustering, spatial visualization, marker genes |
| 2 | [Visium Fluorescence](02_visium_fluorescence/) | Mouse Brain (Visium + fluorescence) | Nucleus segmentation, image-derived features |
| 3 | [Visium H&E](03_visium_hne/) | Mouse Brain (Visium + H&E) | Neighborhood enrichment, co-occurrence, Moran's I |
| 4 | [Xenium](04_04_xenium/) | Human Lung Cancer (Xenium) | Single-cell spatial, centrality scores, co-occurrence |

Each folder contains its own README with a full step-by-step workflow and annotated results.

---

## Key Findings

### Section 1 — Basic Scanpy
Approximately 4,000 Visium spots from a human lymph node were processed through a standard scRNA-seq pipeline. Leiden clustering revealed a strong correspondence between transcriptional identity and the physical H&E image. Cluster 9 was identified as a follicular region, confirmed by high expression of the marker gene *CR2*. This demonstrates that tissue architecture can be computationally recovered without manual pathologist annotation.

### Section 2 — Visium Fluorescence
DAPI and antibody staining (NEUN/GFAP) were used to segment cell nuclei via Watershed and extract per-spot image features. When spots were clustered on image features alone — with no sequencing data — the hippocampus was subdivided into known anatomical sub-layers that gene expression clusters had grouped as a single undifferentiated block. This shows that morphological information can resolve structure beyond what transcriptomics captures alone.

### Section 3 — Visium H&E
Neighborhood enrichment analysis (permutation test) confirmed that the Pyramidal layer and Hippocampus are statistically significant neighbors in the tissue. Moran's I identified genes such as *Olfm1* and *Plp1* as having highly non-random spatial distributions. Ligand-receptor screening identified candidate intercellular communication pairs at cluster boundaries.

### Section 4 — Xenium
Over 11,000 individual lung cancer cells were analyzed at single-cell resolution. Centrality scores computed on a Delaunay triangulation graph identified which cell types act as structural hubs in the tumor microenvironment. Spatial co-occurrence analysis showed that tumor cells form tight, high-density networks clearly distinct from the surrounding healthy stroma.

---

## Definitions

**Spots** — Physical circles on the Visium glass slide, 55 µm in diameter. Each spot captures RNA from a small group of 1–10 cells and acts as a single pixel of gene expression data.

**Clusters** — Groups of spots or cells sharing similar gene expression profiles, identified by the Leiden algorithm based on a neighborhood graph.

**AnnData** — The standard data structure for single-cell and spatial analysis in Python. Stores gene counts, metadata, and spatial coordinates in a single object.

**H&E Staining** — Hematoxylin and Eosin staining. Hematoxylin (purple) stains cell nuclei; Eosin (pink) stains proteins and extracellular structures. Provides a visual map of tissue morphology.

**PCA & UMAP** — Dimensionality reduction tools. PCA compresses thousands of gene dimensions into principal components; UMAP further reduces these to a 2D embedding that preserves local structure, making clusters visually distinguishable.

**Moran's I** — A spatial autocorrelation statistic ranging from −1 to 1. A high positive score indicates that a gene's expression is spatially clustered — forming patterns rather than being distributed randomly across the tissue.

**Ligand–Receptor Pairs** — A ligand is a signaling molecule secreted by one cell; a receptor is the complementary protein on a neighboring cell that detects it. Identifying co-expressed pairs across cluster boundaries reveals active intercellular communication.

**Watershed Segmentation** — A classical image segmentation algorithm. The image intensity landscape is treated as a topographic surface; the algorithm floods from local minima until regions merge at boundaries, effectively separating touching nuclei.

**Delaunay Triangulation** — A method for constructing a spatial neighbor graph by connecting cell centers into triangles such that no cell falls inside the circumcircle of any triangle. Produces a physically meaningful definition of neighbor for irregularly positioned single cells.

---

## Tools & Libraries

| Library | Role |
|---------|------|
| Scanpy | Single-cell preprocessing, clustering, UMAP |
| Squidpy | Spatial statistics, image features, graph analysis |
| SpatialData / Zarr | Xenium data loading and high-performance storage |
| Matplotlib / Seaborn | Visualization |
