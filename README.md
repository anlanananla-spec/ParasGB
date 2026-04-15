# ParasGB Supplementary Appendix Repository

This repository is a standalone appendix-style companion for the paper **ParasGB: A Graph Benchmark Suite for Parasitic Estimation on AMS Circuits**. It is designed to host the supplementary materials that are difficult to keep in the main manuscript under page limits, especially figures, tables, extended visualizations, and short appendix notes.

The scope of this repository is intentionally narrow. It is **not** the training codebase, **not** the full dataset release portal, and **not** a replacement for the paper itself. Instead, it serves as a clean and stable home for the appendix content extracted and reorganized from the `24059` version of the manuscript.

## Repository Goal

The `24059` paper version contains a richer appendix than the conference-length manuscript can accommodate. Those appendix materials are valuable because they provide:

- detailed dataset statistics for analog and SRAM subsets,
- full label-distribution figures,
- additional feature-definition tables,
- extended benchmark result tables,
- preprocessing and normalization notes,
- graph-construction illustrations,
- and the definition of effective-resistance supervision.

This repository isolates those appendix materials into a dedicated GitHub repository so that readers can browse them directly without searching through the PDF.

## What This Repository Contains

The supplementary content is centered around four kinds of artifacts:

- **Figures**  
  High-resolution appendix figures, including label histograms, feature visualizations, conversion diagrams, and workflow illustrations.

- **Tables**  
  Appendix tables for dataset statistics, feature definitions, task settings, and extended benchmark results.

- **Short appendix notes**  
  Lightweight Markdown documents that explain how to read the figures and tables, and provide context that is too detailed for the main paper.

- **Metadata and manifests**  
  Optional small files that map figure/table names to paper sections, appendix labels, and released assets.

## Appendix Coverage

This repository is based on the appendix-oriented content of the `24059` manuscript. A practical way to organize the materials is by appendix topic rather than by raw paper figure number.

### Appendix A: Usage and Benchmark Interface

This section can host supporting materials for benchmark usage, such as:

- a compact workflow figure for loading datasets and evaluators,
- screenshots or diagrams for the benchmark interface,
- a small table summarizing task names and metrics,
- a short user-guide page if a code release is added later.

### Appendix B: Dataset Statistics and Feature Description

This is one of the most important sections for a supplementary repository. It should collect:

- analog dataset statistics,
- SRAM dataset statistics,
- circuit-scale descriptions,
- feature-definition tables,
- label-distribution figures,
- and any per-circuit or per-subset data summary panels.

Typical materials from the appendix include:

- `Table 10`: SRAM graph node feature definitions,
- detailed distribution figures for analog and SRAM targets,
- extended descriptions of analog and SRAM circuit families.

### Appendix C: Preprocessing, Normalization, and Experimental Notes

This section is the right place for appendix-only support materials such as:

- normalization workflow figures,
- class discretization notes,
- evaluator and metric conventions,
- split summaries,
- common hyperparameter tables,
- hardware or implementation notes.

This section helps the repository remain useful even when the main paper can only keep the most concise version of the benchmark protocol.

### Appendix D: Extended Benchmark Results

This section should collect result tables that are useful but too large for the main paper layout. In the `24059` appendix, that includes extra regression and classification tables that expand the benchmark story beyond the headline results.

Recommended contents:

- analog task result tables,
- SRAM task result tables,
- regression-only supplementary results,
- per-task tables that were omitted for space reasons.

Examples of appendix-style result tables that fit naturally here:

- SRAM ground-capacitance node regression,
- analog ground-capacitance node regression,
- SRAM coupling-capacitance edge regression,
- analog effective-resistance edge regression,
- SRAM effective-resistance classification,
- and omitted classification tables from the longer manuscript version.

### Appendix E: Limitations

If you want the repository to feel complete as a standalone appendix companion, this section can include:

- a short note on current benchmark scope,
- what is not released due to confidentiality,
- process-node or circuit-family coverage limits,
- and benchmark-design tradeoffs.

### Appendix F: Future Directions

This section can hold a short forward-looking note about:

- broader circuit-family coverage,
- richer physical modeling,
- geometry-aware learning,
- foundation-model directions,
- or future benchmark extensions.

### Appendix G: Analog-Specific Graph Construction

This section is well suited for visual materials. It should host:

- the analog graph-conversion figure,
- a step-by-step schematic-to-graph explanation,
- a node/edge schema table,
- and optional pseudocode or export notes.

This is where readers can understand how design data and extracted parasitic information are turned into the learnable heterogeneous graphs used in ParasGB.

### Appendix H: Effective Resistance (`Reff`) Definition

This section should explain the `Reff` target clearly and visually. Good supplementary assets here include:

- a workflow diagram for `Reff` generation,
- a notation table,
- a short algorithm note,
- and a compact explanation of the reduced admittance matrix formulation.

For many readers, this appendix is essential because `Reff` is not a trivial label copied directly from a single resistor element.

## Suggested Repository Layout

```text
.
├── README.md
├── docs/
│   ├── appendix_A_usage.md
│   ├── appendix_B_dataset_statistics.md
│   ├── appendix_C_preprocessing.md
│   ├── appendix_D_extended_results.md
│   ├── appendix_E_limitations.md
│   ├── appendix_F_future_work.md
│   ├── appendix_G_graph_construction.md
│   └── appendix_H_reff_definition.md
├── figures/
│   ├── paper/
│   └── appendix/
│       ├── dataset_statistics/
│       ├── label_distributions/
│       ├── preprocessing/
│       ├── graph_construction/
│       ├── reff_definition/
│       └── extra_results/
├── tables/
│   ├── analog/
│   ├── sram/
│   └── benchmark/
├── metadata/
│   ├── figure_manifest.csv
│   ├── table_manifest.csv
│   └── appendix_mapping.md
└── assets/
    └── thumbnails/
```

## Recommended Content Mapping

If you want to populate the repository directly from the `24059` appendix, the following mapping is a practical starting point:

| Appendix topic | Recommended repository location |
|---|---|
| dataset statistics and feature definitions | `docs/appendix_B_dataset_statistics.md`, `tables/analog/`, `tables/sram/` |
| preprocessing and normalization | `docs/appendix_C_preprocessing.md`, `figures/appendix/preprocessing/` |
| omitted benchmark result tables | `docs/appendix_D_extended_results.md`, `tables/benchmark/` |
| analog graph conversion | `docs/appendix_G_graph_construction.md`, `figures/appendix/graph_construction/` |
| `Reff` algorithm and notation | `docs/appendix_H_reff_definition.md`, `figures/appendix/reff_definition/`, `tables/benchmark/` |
| label histograms and feature-space figures | `figures/appendix/label_distributions/` |

## Key Appendix Assets to Prioritize

If the goal is to build a useful GitHub repository quickly, the highest-value materials to upload first are:

1. full label-distribution figures for analog and SRAM tasks,
2. SRAM feature-definition table,
3. supplementary regression and classification result tables,
4. analog graph-conversion figure,
5. `Reff` definition figure and notation table,
6. one dataset-statistics table each for analog and SRAM.

These materials carry most of the information that is usually cut from the main paper for page-limit reasons.

## Suggested File Naming

Use stable descriptive names instead of raw manuscript screenshots. A consistent naming scheme makes the repository much easier to browse and maintain.

Recommended conventions:

- figures: `fig_<appendix>_<topic>_<subset>.<ext>`
- tables: `table_<appendix>_<topic>_<subset>.<ext>`
- docs: `appendix_<letter>_<topic>.md`

Examples:

- `fig_B_analog_label_distribution.png`
- `fig_B_sram_cc_distribution.png`
- `fig_G_analog_graph_conversion.png`
- `fig_H_reff_algorithm_overview.png`
- `table_B_sram_feature_definition.csv`
- `table_D_sram_cg_node_regression.csv`
- `table_D_analog_reff_edge_regression.csv`

## Reading Guide

Readers do not need to open every file in order. A compact reading path is:

1. read the main paper for the benchmark motivation and core contributions,
2. open Appendix B materials for dataset coverage and feature definitions,
3. open Appendix D tables for the extended results,
4. use Appendix G and H for the two most technical construction details,
5. refer to Appendix C when reproduction details are needed.

## What This Repository Does Not Try to Do

To keep the repository focused, it should not try to become:

- the official training framework,
- a full benchmark website,
- a replacement for the main manuscript,
- or a dump of every internal artifact from the design flow.

Its role is simpler: it is an appendix repository for figures, tables, and concise supplementary explanations.

## Anonymity Note

If this repository is used during anonymous review, keep it neutral:

- do not include author names,
- do not add personal GitHub links,
- do not include institutional logos,
- do not expose identifying acknowledgements,
- and avoid metadata that reveals authorship.

## Citation

You can keep a placeholder citation during review and replace it after acceptance:

```bibtex
@article{parasgb,
  title   = {ParasGB: A Graph Benchmark Suite for Parasitic Estimation on AMS Circuits},
  author  = {Anonymous Authors},
  year    = {2026}
}
```

## Current Status

This repository currently serves as the entry point for reorganizing the appendix content from the `24059` manuscript into a standalone supplementary GitHub archive. The next step is to populate the `figures/`, `tables/`, and `docs/` folders with the final exported appendix assets.
