---
title: 'Athlytics: Reproducible Scientific Workflows for Cohort Analysis of Endurance Training Using Local Strava Exports'
tags:
  - R
  - sports science
  - endurance
  - Strava
  - cohort analysis
  - reproducibility
  - ACWR
  - cardiovascular decoupling
authors:
  - name: Zhiang He
    orcid: 0009-0009-0171-4578
    affiliation: 1
affiliations:
  - name: Medical Artificial Intelligence Innovation Center, Shanghai East Hospital, Tongji University, Shanghai, China
    index: 1
date: 13 June 2026
bibliography: paper.bib
version: 1.0.5
license: MIT
---

# Summary

Athlytics is an R package for reproducible, offline analysis of endurance-training data exported from Strava. It reads local ZIP and CSV exports and provides a pipeline for import, quality control, cohort summaries, and visualization of metrics including acute-to-chronic workload ratios (ACWR) [@gabbett2016], aerobic efficiency, and cardiovascular decoupling (pa:hr). The package is designed for analyses that need to be repeatable without OAuth credentials, API quotas, or changing service availability.

# Statement of Need

Researchers and coaches working with wearable data often need to combine API clients, file parsers, quality-control scripts, and metric-specific code before analysis can begin. This is especially fragile for cohort-scale studies, where authentication, rate limits, and inconsistent export formats can obstruct reproducible workflows. Athlytics fills this gap by offering one research-oriented R workflow for local Strava export archives, from raw XML/FIT/CSV-derived activity data to physiological indicators and cohort reference bands. Its primary audience is sports scientists, sports epidemiologists, and endurance coaches who need auditable, programmatic analyses across one or more athletes.

# Related Work

We provide a direct feature comparison to highlight the capabilities essential for reproducible, cohort-scale research.

| Feature (research-relevant) | **Athlytics** | rStrava [@rStrava] | trackeR [@trackeR_jss] | activatr [@activatr] | ACWR [@ACWR] | injurytools [@injurytools] |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **Offline archives; No OAuth/tokens/quotas** | ✓ | ✕ (API) | ✓ | ✓ | ✓ (tabular) | ✓ (tabular) |
| **API-limited (OAuth, scope, rate-limits)** | ✕ | ✓ | ✕ | ✕ | ✕ | ✕ |
| **End-to-end pipeline (Import→QC→Models→Plot)** | ✓ | ✕ | **Partial** (parsing/viz) | **Partial** (parsing/pace) | ✕ | ✕ |
| **Built-in metrics (ACWR/EF/decoupling)** | ✓ | ✕ | ✕ | ✕ | **Partial** (ACWR only) | ✕ |
| **Steady-state guards & HR-coverage checks** | ✓ | ✕ | ✕ | ✕ | ✕ | ✕ |
| **Uncertainty (ACWR-EWMA confidence bands)** | ✓ | ✕ | ✕ | ✕ | ✕ | ✕ |
| **Cohort benchmarking (percentile bands)** | ✓ | ✕ | **Partial** (summaries only) | ✕ | ✕ | **Partial** (for injury/exposure) |
| **Diagnostic outputs (status codes/fields)** | ✓ | ✕ | ✕ | ✕ | ✕ | ✕ |

Compared with existing R tools, Athlytics combines local Strava export ingestion, quality-control checks, ACWR/EF/decoupling workflows, uncertainty summaries, and cohort reference bands in a single offline workflow.

# Software Description

-   **Offline Data Parsing:** Operates directly on local Strava ZIP exports. Using `.tcx` and `.gpx` parsers through `xml2` plus optional `.fit` parsing through `FITfileR` [@FITfileR], activity streams are loaded on demand.
-   **Physiological & Load Metrics:** Supports multiple load tracking algorithms including HRSS (TRIMP-based) and TSS approximations. Calculates core metrics such as cardiovascular decoupling (pa:hr), Efficiency Factor (EF), and automatically tracks Personal Bests (PBs) using sliding-window spatial algorithms.
-   **Signal Processing & Quality Control:** Automatically filters implausible HR, power, and velocity samples and identifies steady-state output segments using a rolling coefficient of variation (CV) algorithm to support valid physiological comparisons.
-   **Uncertainty Quantification:** Provides confidence intervals for EWMA-based ACWR models using a moving-block bootstrap [@kunsch1989], partly preserving short-range temporal dependence in training loads. This uncertainty reporting acknowledges the ongoing conceptual debates surrounding ACWR as a predictive tool [@impellizzeri2020acwr; @impellizzeri2021dismiss].
-   **Cohort Benchmarking & Visualization:** Generates population-level percentile reference bands (`calculate_cohort_reference()`) that can be layered onto publication-ready ACWR plots (e.g., `plot_acwr_enhanced()`).
-   **Diagnostics & Transparency:** Functions return **diagnostic fields** (e.g., `status`, `quality_score`, `hr_coverage`) when inputs are insufficient, making the workflow transparent and debuggable.

# Example

The following example demonstrates a common cohort analysis workflow: using pre-computed sample ACWR data to construct a synthetic three-athlete cohort, deriving cohort-wide reference bands, and plotting one athlete against those bands.

```r
library(Athlytics)
library(dplyr)

# 1. Use built-in sample data to simulate a cohort of athletes
data("sample_acwr", package = "Athlytics")
set.seed(42)
cohort_acwr <- bind_rows(
  sample_acwr %>% mutate(athlete_id = "A1"),
  sample_acwr %>% mutate(
    athlete_id = "A2", 
    acwr_smooth = acwr_smooth * runif(n(), 0.9, 1.1)
  ),
  sample_acwr %>% mutate(
    athlete_id = "A3", 
    acwr_smooth = acwr_smooth * runif(n(), 0.85, 1.15)
  )
)

# 2. Generate cohort-wide percentile reference bands
reference_bands <- calculate_cohort_reference(
  cohort_acwr, 
  metric = "acwr_smooth", 
  by = character(0),
  min_athletes = 2
)

# 3. Extract individual data
individual_acwr <- cohort_acwr %>% filter(athlete_id == "A1")

# 4. Plot an individual's ACWR against the cohort reference using the enhanced plotting API
plot_acwr_enhanced(
  acwr_data = individual_acwr,
  reference_data = reference_bands,
  show_ci = FALSE,
  show_reference = TRUE,
  highlight_zones = TRUE
)
```

# Acknowledgements

The author thanks Benjamin S. Baumer and Iztok Fister Jr. for their insightful feedback during early development, and rOpenSci handling editor Emily Riederer and reviewers Eunseop Kim and Simon Nolte for their peer-review guidance.

This work received no external funding.

# References
