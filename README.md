# ParasGB Supplementary Materials

This repository contains the **supplementary materials** for the paper:

**ParasGB: A Graph Benchmark Suite for Parasitic Estimation on AMS Circuits**

It is created for materials that cannot be fully included in the main paper because of page limits, such as:

- appendix figures,
- extended result tables,
- dataset statistics,
- graph-construction details,
- preprocessing notes,
- evaluation details.

> **Important**
> This repository is a **supplementary-material repository**, not the main training-code repository.
> Its purpose is to help reviewers and readers quickly find the extra figures, tables, and notes referenced by the paper.

---

## 1. What is included here?

This repository is intended to organize supplementary content for the following parts of the paper:

- **Dataset statistics** for analog and SRAM subsets
- **Additional figures** that are too large or too numerous for the paper
- **Task definitions** for node-level and edge-level prediction
- **Preprocessing and normalization notes**
- **Graph construction details** from circuit topology / post-layout netlists
- **Extended benchmark results** and ablations

In particular, the repository is suitable for storing materials related to:

- **ground capacitance** (`Cg`)
- **coupling capacitance** (`Cc`)
- **effective resistance** (`Reff`)

and for both:

- **analog circuits**
- **SRAM / AMS-scale circuits**

---

## 2. How to use this repository

If you are reading the paper and want to find additional materials:

- For extra figures, go to **`figures/`**
- For full tables or csv/xlsx files, go to **`tables/`**
- For written explanations, go to **`docs/`**
- For file indexes, go to **`metadata/`**

A simple rule is:

- **visual content** → `figures/`
- **numeric/tabular content** → `tables/`
- **text explanations** → `docs/`
- **file mapping / index** → `metadata/`

---

## 3. Recommended repository structure

```text
.
├── README.md
├── docs/
│   ├── appendix.pdf
│   ├── supplementary_notes.md
│   ├── appendix_B_dataset_statistics.md
│   ├── appendix_C_preprocessing.md
│   ├── appendix_G_graph_construction.md
│   └── appendix_H_reff_definition.md
├── figures/
│   ├── paper/                         # figures also used in the main paper
│   └── appendix/                      # supplementary-only figures
│       ├── label_distributions/
│       ├── dataset_statistics/
│       ├── feature_visualization/
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

---

## 4. What should be placed in each folder?

### `docs/`
Put explanatory documents here.

Examples:

- appendix text exported as PDF
- notes describing how each figure/table should be read
- details that are referenced in the paper but omitted from the main text
- definitions of labels, metrics, splits, and preprocessing steps

Suggested files:

- `appendix_B_dataset_statistics.md`
- `appendix_C_preprocessing.md`
- `appendix_G_graph_construction.md`
- `appendix_H_reff_definition.md`

### `figures/paper/`
Put figures that appear in the main paper here.

Purpose:

- keep the paper figures in one place,
- provide high-resolution versions for zooming,
- make it easy to cross-check paper numbering.

### `figures/appendix/`
Put supplementary-only figures here.

Suggested subfolders:

- `label_distributions/` — histogram / long-tail plots for `Cg`, `Cc`, `Reff`
- `dataset_statistics/` — size/statistics plots for analog and SRAM datasets
- `feature_visualization/` — t-SNE or feature-distribution figures
- `graph_construction/` — topology-to-graph conversion diagrams
- `extra_results/` — extra performance plots, ablations, error analysis

### `tables/analog/`
Put tables for the analog subset here.

Examples:

- analog dataset statistics
- analog node-level results
- analog edge-level results
- analog ablation tables

### `tables/sram/`
Put tables for the SRAM subset here.

Examples:

- SRAM dataset statistics
- SRAM node-level results
- SRAM edge-level results
- cross-scale evaluation tables

### `metadata/`
Use this folder as an index.

This folder is very useful when the repository grows larger.
Each row can record:

- file name,
- related section/appendix,
- paper figure/table number,
- short description.

### `assets/`
Use this for small support files such as thumbnails or lightweight preview assets.

---

## 5. Suggested mapping from the paper to this repository

A clear way to organize the repository is to mirror the appendix structure of the paper.

| Paper part | Suggested location in repo |
|---|---|
| Appendix B: dataset statistics | `docs/appendix_B_dataset_statistics.md` + `figures/appendix/dataset_statistics/` + `tables/analog/`, `tables/sram/` |
| Appendix C: preprocessing / normalization | `docs/appendix_C_preprocessing.md` |
| Appendix G: graph construction | `docs/appendix_G_graph_construction.md` + `figures/appendix/graph_construction/` |
| Appendix H: effective resistance details | `docs/appendix_H_reff_definition.md` |
| Extra benchmark results | `figures/appendix/extra_results/` + `tables/analog/` + `tables/sram/` |

This makes the repository much easier to navigate because readers can directly jump from a paper appendix section to the corresponding folder.

---

## 6. Naming convention

Use consistent file names so reviewers can immediately understand what each file contains.

Recommended format:

```text
fig_<topic>_<subset>.<ext>
table_<topic>_<subset>.<ext>
note_<topic>.<ext>
```

Examples:

```text
fig_label_distribution_sram.png
fig_graph_construction_pipeline.pdf
table_analog_dataset_statistics.csv
table_sram_reff_results.xlsx
note_preprocessing.md
```

Good naming should answer two questions directly:

1. **What is this file about?**
2. **Which subset / task does it belong to?**

---

## 7. Minimal example of what to add first

If you want to build this repository quickly, the most useful first batch is:

1. `docs/appendix_B_dataset_statistics.md`
2. `docs/appendix_C_preprocessing.md`
3. `docs/appendix_G_graph_construction.md`
4. `figures/appendix/label_distributions/`
5. `figures/appendix/graph_construction/`
6. `tables/analog/table_analog_dataset_statistics.csv`
7. `tables/sram/table_sram_dataset_statistics.csv`
8. `metadata/figure_manifest.csv`
9. `metadata/table_manifest.csv`

This is usually enough to make the supplementary repository understandable even before all materials are uploaded.

---

## 8. Reviewer-facing checklist

Before sharing the repository, check the following:

- the repository is **anonymous** if double-blind review is required;
- file names are descriptive and stable;
- folders match the appendix or paper structure;
- figures and tables can be found without reading the whole manuscript;
- every major appendix section has a corresponding file or folder;
- broken links are removed;
- identity-revealing metadata is removed if necessary.

---

## 9. Anonymity note

If this repository is used during anonymous review, please avoid:

- author names,
- personal emails,
- institutional logos,
- links to personal homepages,
- links to public profiles that reveal authorship,
- commit history that exposes identities.

You can replace these after the paper is accepted.

---

## 10. Citation

You may later replace the placeholder citation with the final publication information.

```bibtex
@article{parasgb,
  title={ParasGB: A Graph Benchmark Suite for Parasitic Estimation on AMS Circuits},
  author={Anonymous Authors},
  year={2026}
}
```

---

## 11. Current status

This repository is currently a **supplementary-material template**.
You can now fill it with the actual appendix figures, tables, and notes from the paper.
