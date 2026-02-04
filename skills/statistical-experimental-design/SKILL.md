---
name: statistical-experimental-design
description: Plan and analyze experiments with statistical rigor. Use when selecting study designs, choosing statistical tests, calculating power/sample size, or setting hypotheses and analysis plans for scientific or product experiments.
---

# Statistical Experimental Design

## Overview

Define hypotheses, choose the right design, and align analysis with data types and assumptions.

## Workflow

1. **Define the question**: Primary outcome, effect size of interest, and decision criteria.
2. **Select a design**: Between-subjects, within-subjects, factorial, or observational.
3. **Choose the analysis**: Map outcomes to tests (see below).
4. **Estimate power**: Compute sample size for the minimum detectable effect.
5. **Pre-register analysis**: Specify exclusion rules and multiple-comparison handling.

## Test Selection Guide

* **Two groups, continuous outcome**: t-test (or Mann–Whitney for non-normal).
* **Paired measurements**: paired t-test (or Wilcoxon signed-rank).
* **>2 groups**: ANOVA (or Kruskal–Wallis).
* **Categorical outcome**: chi-square or Fisher’s exact.
* **Relationship between variables**: Pearson/Spearman correlation, linear regression.

## Power and Sample Size Notes

* Use conservative effect sizes when uncertain.
* Adjust for multiple comparisons (Bonferroni or FDR).
* For sequential testing, plan alpha spending or Bayesian stopping rules.

## Reporting Checklist

* Define the primary endpoint.
* Report confidence intervals and effect sizes.
* Disclose exclusions and missing data handling.
* Verify assumptions (normality, variance, independence).
