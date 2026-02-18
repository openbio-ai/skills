---
name: omics-pipelines
description: Plan and execute genomics, transcriptomics, proteomics, or metabolomics pipelines. Use when building end-to-end workflows from raw sequencing/mass-spec data through QC, alignment, quantification, and downstream analysis.
---

# Omics Pipelines

## Overview

Guide end-to-end omics workflows with explicit QC, reproducible steps, and clear outputs.

## Pipeline Backbone

1. **Input and metadata**: Confirm file types (FASTQ, BAM, mzML) and sample sheet.
2. **QC and trimming**: Run per-sample QC; trim/adapt as needed.
3. **Alignment/assembly**: Align to reference or assemble de novo.
4. **Quantification**: Generate count matrices or abundance tables.
5. **Normalization**: Apply appropriate scaling and batch correction.
6. **Differential analysis**: Use model-based methods with covariates.
7. **Reporting**: Include QC summaries and reproducible parameters.

## Modality Notes

* **Bulk RNA-seq**: Emphasize mapping rate, gene body coverage, and batch effects.
* **scRNA-seq**: Filter by UMI counts, mitochondrial %, doublets.
* **Proteomics**: Control FDR at peptide/protein level; track missingness.
* **Metabolomics**: Normalize for batch drift; annotate peaks conservatively.

## Common Outputs

* QC reports (per-sample and aggregate)
* Count/abundance matrices
* Differential results with log2FC and adjusted p-values
* Visualization summaries (PCA/UMAP, heatmaps, volcano plots)
