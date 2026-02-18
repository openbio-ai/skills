---
name: data-visualization
description: Create publication-ready visualizations from scientific, product, or business data. Use when turning tabular or time-series data into charts, choosing chart types, formatting plots for papers or dashboards, or exporting figures with consistent styling.
---

# Data Visualization

## Overview

Create clear, accurate charts with appropriate encodings, minimal clutter, and export-ready styling.

## Workflow

1. **Clarify the goal**: Identify the key comparison or trend (rank, distribution, change over time, relationship).
2. **Audit the data**: Confirm units, missing values, categories, and sample size.
3. **Select the chart type**: Use the decision guide below.
4. **Style for readability**: Use consistent fonts, colorblind-safe palettes, and labeled axes.
5. **Export correctly**: Prefer vector formats (SVG/PDF) for publications, PNG for web.

## Chart Selection Guide

* **Trend over time**: Line chart with confidence bands or multiple series.
* **Distribution**: Histogram, violin, or box plot.
* **Category comparison**: Bar chart (sorted) with error bars if appropriate.
* **Relationship**: Scatter plot with regression line or density contours.
* **Composition**: Stacked bars or 100% stacked bars (avoid pie charts unless 2–3 slices).

## Quality Checklist

* Include axis labels with units.
* Avoid dual axes unless absolutely necessary.
* Use consistent scales across panels.
* Annotate key events or thresholds.
* Report sample size (n) in captions or labels.

## Export Tips

* **Publication**: SVG or PDF, 300+ DPI if raster required.
* **Slides/web**: PNG at 2x resolution.
* **Reproducibility**: Save the code and the exact input data snapshot.
