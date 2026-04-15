# ParasGB Supplementary Repository

This repository hosts the supplementary materials for the paper **"ParasGB: A Graph Benchmark Suite for Parasitic Estimation on AMS Circuits"**. It is intended to complement the main manuscript by providing the figures, tables, extended explanations, and appendix-style notes that are difficult to include in full within the page limit.

The repository is organized so that a reader can quickly locate:

- extended dataset statistics for analog and SRAM subsets,
- additional label-distribution figures and feature visualizations,
- preprocessing and normalization details,
- graph-construction notes that explain how topology and parasitic labels are formed,
- the effective-resistance construction used for edge-level `Reff` supervision,
- supplementary tables, thumbnails, and metadata files referenced by the appendix.

## What this repository contains

ParasGB is designed as an open benchmark for post-layout parasitic estimation on circuit graphs. The benchmark covers:

- two circuit families: **analog** circuits and **SRAM / AMS-scale** circuits,
- two prediction granularities: **node-level** and **edge-level** tasks,
- three parasitic targets: **ground capacitance (`Cg`)**, **coupling capacitance (`Cc`)**, and **effective resistance (`Reff`)**,
- two learning settings: **classification** and **regression**.

This supplementary repository is therefore structured around those four axes: data source, graph construction, label preprocessing, and target definition.

## Quick navigation

- `docs/appendix_B_dataset_statistics.md`  
  Extended dataset notes, split summaries, scale definitions, feature descriptions, and pointers to additional histograms / tables.

- `docs/appendix_C_preprocessing.md`  
  Details of preprocessing, normalization, discretization, split conventions, evaluator-related notes, and experimental settings that are too long for the main paper.

- `docs/appendix_G_graph_construction.md`  
  Explanation of how schematic/topology information is converted into heterogeneous graphs, with a dedicated section for analog-specific construction details.

- `docs/appendix_H_reff_definition.md`  
  Explanation of how edge-level effective resistance labels are computed, including notation, assumptions, algorithmic steps, and suggestions for supplementary figures.

## Repository structure

```text
parasgb-supplementary/
├── README.md
├── docs/
│   ├── appendix_B_dataset_statistics.md
│   ├── appendix_C_preprocessing.md
│   ├── appendix_G_graph_construction.md
│   └── appendix_H_reff_definition.md
├── figures/
│   ├── paper/                # figures reused from the paper for cross-reference
│   └── appendix/             # supplementary figures, histograms, workflow diagrams
├── tables/
│   ├── analog/               # csv/xlsx/pdf tables for analog subset
│   └── sram/                 # csv/xlsx/pdf tables for SRAM subset
├── metadata/                 # split files, naming notes, dataset manifests, checksums
└── assets/
    └── thumbnails/           # small preview images used inside markdown pages
```

## Suggested reading order

For readers who only want a fast overview:

1. Read the paper.
2. Open `docs/appendix_B_dataset_statistics.md` for scale, dataset, and feature details.
3. Open `docs/appendix_C_preprocessing.md` for normalization and training/evaluation conventions.
4. Open `docs/appendix_G_graph_construction.md` for the topology-to-graph pipeline.
5. Open `docs/appendix_H_reff_definition.md` for the `Reff` label-generation procedure.

## Appendix-to-repository mapping

| Paper appendix / topic | Repository file |
|---|---|
| Appendix B: dataset statistics, scale coverage, feature definitions | `docs/appendix_B_dataset_statistics.md` |
| Appendix C: preprocessing, normalization, splits, evaluator / settings | `docs/appendix_C_preprocessing.md` |
| Appendix G: analog-specific graph construction | `docs/appendix_G_graph_construction.md` |
| Appendix H: effective resistance definition and algorithm | `docs/appendix_H_reff_definition.md` |

## What you still need to add

The text files in `docs/` are written so they can already be pushed to GitHub. To finish the repository, you mainly need to add the corresponding non-text assets:

- supplementary figures into `figures/appendix/`,
- reusable paper figures into `figures/paper/`,
- extended tables into `tables/analog/` and `tables/sram/`,
- optional manifests / split files into `metadata/`.

In other words, the narrative part is already prepared; the remaining work is mostly to place images, tables, and exported artifacts into the expected folders.

## Recommended file naming

To keep appendix references clean, it is helpful to adopt stable names such as:

- `fig_B1_analog_label_histograms.png`
- `fig_B2_sram_label_histograms.png`
- `fig_C1_normalization_pipeline.png`
- `fig_G1_analog_graph_conversion.png`
- `fig_H1_reff_algorithm_overview.png`
- `table_B1_analog_statistics.csv`
- `table_B2_sram_statistics.csv`

## Anonymity note

If this repository is used during anonymous review, avoid adding:

- author names,
- personal or institutional GitHub profile links,
- acknowledgements,
- project pages that reveal identity,
- commit messages or badges that expose authorship.

A neutral repository title and neutral prose are usually safest for double-blind submission.

## Citation

Please replace the placeholder below with your final citation after the review process:

```bibtex
@article{parasgb,
  title   = {ParasGB: A Graph Benchmark Suite for Parasitic Estimation on AMS Circuits},
  author  = {Anonymous Authors},
  journal = {Under Review},
  year    = {2026}
}
```
