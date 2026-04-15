# ParasGB Supplementary Materials Repository

This repository accompanies the paper **“ParasGB: A Graph Benchmark Suite for Parasitic Estimation on AMS Circuits”** and is intended to serve as a clean, structured home for the paper’s extended materials.

It is **not** a training code repository and **not** a standalone dataset release page. Instead, it collects the figures, tables, and supporting visual materials that provide additional detail beyond the main paper. The goal is to make the extended content easy to browse, easy to cite in rebuttal or revision, and easy to maintain during resubmission.

## Overview

ParasGB is an open benchmark for parasitic prediction on circuit graphs derived from real AMS design flows. The benchmark brings together two complementary circuit families:

- **Analog circuits**, which emphasize fine-grained parasitic behavior in relatively smaller but highly sensitive designs.
- **SRAM circuits**, which emphasize scalability, dense interconnect structure, and extremely large graph sizes.

Across these circuit families, the benchmark covers:

- **Node-level prediction**, mainly for ground capacitance (**Cg**).
- **Edge-level prediction**, including coupling capacitance (**Cc**) and effective resistance (**Reff**).
- **Classification tasks**, where parasitic values are discretized into bins for coarse-grained prediction.
- **Regression tasks**, where the objective is direct numerical prediction.

The main paper introduces the benchmark definition, dataset construction pipeline, task settings, and core classification results. This repository stores the **extended visual and tabular materials** that further explain the benchmark, data characteristics, preprocessing choices, detailed experiments, and additional technical notes.

## What is included here

The content in this repository is organized around the major groups of supporting material associated with the paper.

### 1. Benchmark usage and evaluation support

The paper introduces a unified benchmark interface and evaluator design inspired by OGB-style evaluation. The related supplementary material explains how the benchmark is intended to be used conceptually:

- the benchmark task taxonomy,
- dataset naming and split conventions,
- evaluator usage for different task types,
- the distinction between analog and SRAM task settings,
- and the role of standardized metrics such as Accuracy, F1, MAE, and R².

In this repository, these materials are represented through explanatory figures, screenshots, workflow diagrams, or compact summary tables rather than executable code.

### 2. Dataset construction details

A major portion of the extended material explains how ParasGB is built from real design flows and how the resulting graph data is organized.

This includes:

- circuit sourcing and validation,
- separation of **analog** and **SRAM** subsets,
- scale groupings for each family,
- graph sizes and structural statistics,
- the definition of prediction targets,
- and the relationship between topology edges and parasitic labels.

For analog circuits, the dataset materials summarize the diversity of circuit blocks and the scale ranges covered by the benchmark. For SRAM circuits, they emphasize the much larger graph scale and the very dense coupling-capacitance edge sets that make these tasks computationally demanding.

### 3. Label distribution materials

The extended materials also provide a much more detailed view of label distributions than can fit in the main text.

These materials include visual overviews of:

- **analog ground capacitance distributions**,
- **analog effective resistance distributions**,
- **SRAM ground capacitance distributions**,
- **SRAM coupling capacitance distributions**,
- and **SRAM effective resistance distributions**.

These distribution plots are important because the benchmark is deliberately challenging: many tasks exhibit long-tailed, highly imbalanced, or sparse label behavior. The corresponding figures and tables in this repository are meant to help readers understand why some tasks are easy in classification but hard in regression, why minority-region prediction remains difficult, and why industrial parasitic data creates distinctive learning challenges.

### 4. Preprocessing and experimental configuration

Another group of extended materials documents how raw values are filtered, normalized, and converted into benchmark-ready prediction targets.

These materials cover:

- valid value filtering for each task,
- task-specific normalization rules,
- regression target scaling,
- class discretization settings,
- baseline model families,
- training configuration summaries,
- and hardware or compute-related settings used for reproducibility.

This part is especially important because analog and SRAM tasks do not use exactly the same target ranges or preprocessing boundaries. The repository therefore keeps these settings visible and separated from the main narrative of the paper.

### 5. Extended benchmark results

The main paper focuses on the central benchmark story, while the supplementary materials include broader result coverage.

In particular, the extended materials record:

- additional **regression results** beyond the main text,
- task-by-task result tables,
- comparisons across node-level and edge-level settings,
- and short comparative observations about why some architectures behave differently on capacitance and resistance tasks.

These materials are useful for readers who want to inspect performance more closely, compare classification and regression behavior, or understand which settings remain hardest for current GNN models.

### 6. Limitations and future directions

The repository also includes materials that discuss the current boundaries of the benchmark and how it may evolve.

These materials describe limitations such as:

- incomplete process-node coverage,
- restricted diversity of circuit categories relative to the full IC design space,
- and limited modeling depth for richer physical effects such as deeper spatial or electromagnetic interactions.

They also summarize future directions, including:

- expanding to broader circuit families,
- improving spatial-geometry-aware modeling,
- exploring graph foundation models and pretraining,
- and moving toward tighter integration with practical design workflows.

### 7. Additional graph-construction explanation

The main paper presents the overall topology-to-graph conversion pipeline, while the extended materials further explain the graph construction procedure for analog circuits in more detail.

This part clarifies:

- how schematics and post-layout netlists are translated into heterogeneous graphs,
- the roles of device, pin, and net nodes,
- how topology edges differ from parasitic supervision edges,
- and how simplified graph representations preserve useful physical structure while keeping learning tractable.

The repository stores the corresponding diagrams and any enlarged versions of these visuals.

### 8. Effective-resistance label generation note

Effective resistance is one of the most technically specific targets in the benchmark, and its construction benefits from a dedicated explanation.

The supporting materials therefore include:

- the rationale behind effective-resistance labels,
- a matrix-based calculation description,
- the use of an invertible admittance matrix,
- Cholesky-based computation steps,
- and the procedure for generating port-to-port resistance labels efficiently.

These materials are intended to make the resistance-target construction more transparent, especially for readers who are familiar with EDA workflows but want to understand how the benchmark operationalizes this signal for learning.

## How to read this repository

A reader does not need to go through every file in order.

A practical reading path is:

1. Start from the main paper for the benchmark motivation, task definition, and headline results.
2. Use this repository when you want the extended figures and tables behind the benchmark story.
3. Look at the dataset and distribution materials first if you want to understand the structure and difficulty of the data.
4. Look at the preprocessing and experiment materials next if you want to reproduce the benchmark setup or compare against it fairly.
5. Look at the graph-construction and effective-resistance materials if you want to understand how the targets are generated from circuit representations.
6. Look at the extended result tables, limitation notes, and forward-looking discussion for a fuller research picture.

## Repository organization

This repository is intentionally lightweight. The files here are meant to support the paper rather than replace it.

```text
.
├── README.md                 # Main guide to the supplementary contents
├── figures/
│   ├── paper/                # Figures already referenced directly in the paper, if mirrored here
│   └── appendix/             # Additional or extended figures supporting the paper
├── tables/
│   ├── analog/               # Tables related to analog circuits
│   └── sram/                 # Tables related to SRAM circuits
├── metadata/                 # Optional split summaries, file manifests, naming records, or notes
├── assets/
│   └── thumbnails/           # Optional preview images for browsing convenience
└── docs/                     # Optional short textual notes accompanying specific figure/table groups
```

### Folder purpose

**`figures/paper/`**
Stores copies of figures that are already discussed in the paper but are useful to mirror here for easy browsing, especially when higher-resolution versions are needed.

**`figures/appendix/`**
Stores the extended visual materials that support the paper, such as label-distribution panels, enlarged workflow diagrams, graph-construction illustrations, and technical schematics that are too detailed or too numerous for the main manuscript.

**`tables/analog/`**
Stores supplementary tables specific to the analog subset, including circuit statistics, task-specific summaries, and extended benchmark results relevant to analog designs.

**`tables/sram/`**
Stores supplementary tables specific to the SRAM subset, including large-scale dataset summaries, edge-label statistics, and extended benchmark results relevant to SRAM experiments.

**`metadata/`**
Stores small machine-readable or human-readable support files, such as file manifests, naming conventions, task-index notes, or mapping files that connect figure/table names to sections of the paper.

**`docs/`**
Stores short companion notes only when a figure or table group needs a little extra explanation. The README remains the main entry point, while the `docs/` folder can be used for targeted technical clarifications.

## What this repository does not try to do

To keep the scope clear, this repository is intentionally limited.

It does **not** aim to:

- serve as the full benchmark toolkit,
- duplicate the main manuscript,
- replace a formal dataset release,
- provide training scripts or environment setup,
- or function as a long narrative document independent of the paper.

Its role is narrower and more practical: it is the structured home for the **supporting figures, tables, and concise explanatory notes** that make the paper easier to inspect.

## Suggested file types to place here

The repository is best suited for:

- figure panels in `.png`, `.jpg`, or `.pdf`,
- benchmark tables in `.csv`, `.xlsx`, or exported `.pdf`,
- compact supplementary notes in `.md`,
- and small metadata files such as manifests or naming maps in `.json`, `.yaml`, or `.txt`.

## Suggested naming style

A consistent naming scheme makes the repository much easier to use. A practical convention is to group files by content instead of by manuscript appendix label.

Examples:

- `figures/appendix/analog_cg_distribution_all_cases.png`
- `figures/appendix/analog_reff_distribution_all_cases.png`
- `figures/appendix/sram_cc_distribution_all_cases.png`
- `figures/appendix/analog_graph_construction.png`
- `tables/analog/analog_dataset_statistics.csv`
- `tables/sram/sram_dataset_statistics.csv`
- `tables/sram/sram_edge_regression_results.csv`
- `metadata/file_manifest.csv`

This keeps the repository readable even if the paper structure changes during resubmission.

## Relationship to the paper

The main manuscript should be treated as the primary scientific narrative.

This repository is best understood as the **extended reference layer** for that narrative. When the paper mentions that additional details are available, those details can be stored here in a cleaner, more inspectable form. In that sense, this repository functions as a companion archive for:

- extended dataset statistics,
- detailed label-distribution panels,
- preprocessing notes,
- full result tables,
- graph-construction illustrations,
- and technical notes on label generation.

## Anonymity note

If this repository is used during anonymous review, all materials should remain anonymized.

Please avoid including:

- author names,
- institution names,
- personal GitHub identifiers,
- internal project paths,
- links that reveal identity,
- or figure/table watermarks that disclose authorship.

A neutral public-facing repository title and neutral file naming are recommended until the review process is complete.

## Citation

If the paper is accepted and the repository remains public, a standard citation block for the final paper can be added here.

For the anonymous review stage, a placeholder citation or no citation block at all is usually safer.

## Final note

This repository is designed so that a reader can quickly understand **what supplementary content exists, why it matters, and where to find it**, without having to infer structure from scattered appendix references.

Once the corresponding figures and tables are added, this README should be sufficient as the main guide for browsing the extended materials.
