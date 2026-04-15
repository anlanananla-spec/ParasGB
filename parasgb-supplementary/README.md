# ParasGB Supplementary Materials

Supplementary repository for the paper **"ParasGB: A Graph Benchmark Suite for Parasitic Estimation on AMS Circuits"**.

This repository is intended to host appendix-style materials that are omitted from the main manuscript due to page limits, including extended figures, additional tables, implementation notes, and extra experimental results.

> **Anonymous-review note**  
> If the submission is under double-blind review, please keep the repository anonymized and avoid adding author names, personal emails, institutional logos, commit history that reveals identity, or public links tied to personal profiles.

## Overview

ParasGB studies parasitic estimation on circuit graphs and focuses on realistic analog and SRAM-scale designs. The supplementary repository is organized so that reviewers and readers can quickly find:

- extended visualizations used to support the main paper,
- appendix tables and statistical summaries,
- graph-construction and preprocessing details,
- extra benchmark results that do not fit in the paper,
- archival assets for future public release.

## Suggested Contents

### 1. Extended Figures
Store high-resolution figures that support the main paper, for example:

- label distribution plots,
- feature-space visualizations,
- topology-to-graph conversion diagrams,
- qualitative comparisons across benchmarks,
- additional result plots for analog and SRAM subsets.

### 2. Supplementary Tables
Use this section for materials such as:

- full dataset statistics,
- per-split benchmark results,
- ablation summaries,
- implementation settings,
- extended metric breakdowns.

### 3. Notes and Documentation
This section can contain:

- preprocessing details,
- graph construction notes,
- task definitions,
- evaluation protocol details,
- clarification for figures and tables in the appendix.

## Repository Layout

```text
.
├── README.md
├── docs/
│   ├── appendix.pdf
│   └── supplementary_notes.md
├── figures/
│   ├── paper/                         # figures used in the main paper
│   └── appendix/                      # appendix-only figures
│       ├── label_distributions/
│       ├── dataset_statistics/
│       ├── graph_construction/
│       └── extra_results/
├── tables/
│   ├── analog/
│   └── sram/
├── metadata/
│   ├── figure_manifest.csv
│   └── table_manifest.csv
└── assets/
    └── thumbnails/
```

## Recommended Naming Rules

For cleaner long-term maintenance, use descriptive file names such as:

- `fig_label_distribution_sram.png`
- `fig_graph_conversion_pipeline.pdf`
- `table_analog_dataset_statistics.csv`
- `table_sram_edge_results.xlsx`
- `supplementary_notes_graph_construction.md`

A simple convention is:

```text
fig_<topic>_<subset>.<ext>
table_<topic>_<subset>.<ext>
note_<topic>.<ext>
```

## How to Populate This Repository

1. Put manuscript figures that also appear in the paper under `figures/paper/`.
2. Put appendix-only visual materials under `figures/appendix/`.
3. Save large tables in `tables/` and keep light-weight previews in PDF, CSV, or Markdown when possible.
4. Track figure/table locations in `metadata/figure_manifest.csv` and `metadata/table_manifest.csv`.
5. Keep explanations in `docs/supplementary_notes.md` so readers can understand each asset without opening the manuscript first.

## Minimal Release Checklist

Before sharing the repository, check the following:

- remove identity-revealing metadata if anonymity is required;
- verify that figure numbers and table numbers match the manuscript;
- ensure all linked files open correctly;
- confirm that filenames are stable and descriptive;
- include only materials that are safe to disclose.

## Citation

If you later make this repository public, you can cite the corresponding paper here.

```bibtex
@article{parasgb,
  title={ParasGB: A Graph Benchmark Suite for Parasitic Estimation on AMS Circuits},
  author={Anonymous Authors},
  year={2026}
}
```

## Contact

For review-time releases, keep this section anonymous. After acceptance, you can replace it with the project homepage, code repository, or author contact information.
