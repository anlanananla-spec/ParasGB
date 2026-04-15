# Appendix B — Dataset Statistics and Feature Description

## Purpose of this document

This page collects the dataset-oriented supplementary material for ParasGB. It is intended to accompany the main paper by giving a fuller description of the benchmark coverage, the two circuit families, the scale taxonomy, the label inventory, and the feature definitions used in graph construction.

The goal of this appendix is not to repeat the full paper word-for-word. Instead, it provides a reader-friendly landing page for the additional tables and figures that should be stored in the repository.

---

## B.1 Benchmark scope

ParasGB is a benchmark for **post-layout parasitic estimation** on circuit graphs. The suite covers two major circuit groups:

### Analog subset

The analog portion contains tape-out-oriented AMS modules such as:

- LVDS,
- BGR,
- OP,
- LDO.

These graphs are relatively smaller than the SRAM subset, but they are still challenging because the target parasitics are extremely small in magnitude and fine-grained physical variation matters.

### SRAM subset

The SRAM portion contains much larger industrial-style designs. Compared with analog blocks, these graphs stress scalability much more strongly because of the dramatic growth in node count, edge count, and the number of parasitic targets—especially coupling-capacitance edges.

---

## B.2 Scale definitions

ParasGB organizes the two circuit groups into different scale bins.

### Analog scales

The analog subset is divided into:

- **XS**
- **S**
- **M**

This partition is intended to reflect the progression from micro-scale analog blocks to more complex analog modules.

### SRAM scales

The SRAM subset is divided into:

- **L**
- **XL**
- **XXL**

This partition reflects the fact that SRAM graphs can span from moderately large designs to extremely large industrial-scale networks.

### Practical note

In the main paper, scale is defined according to the number of net nodes, because the size of the circuit matrix and the cost of simulation are strongly tied to net-level complexity. When you add your final table or figure here, it is useful to restate the exact scale threshold used in the released dataset.

**Suggested insertion point:**
- add a compact table here listing the exact thresholds or naming convention used to map each dataset into its scale category.

---

## B.3 What to include in the supplementary figures and tables

This appendix is the natural place to store the following materials:

### Recommended figures

1. **Per-circuit label histograms for analog graphs**  
   For example, histogram panels for `Cg` and `Reff` over all analog designs.

2. **Per-dataset label histograms for SRAM graphs**  
   For example, histogram panels for `Cg`, `Cc`, and `Reff` on `digtime`, `timing_ctrl`, `array_128`, `sram`, `ultra8t`, and `sandwich`.

3. **Feature-space visualizations**  
   Any expanded t-SNE / UMAP / PCA visualizations beyond what appears in the paper.

4. **Cross-scale comparisons**  
   Figures that show how the label range, node count, and edge count evolve from XS to M or from L to XXL.

### Recommended tables

1. **Full analog statistics table**  
   Include node counts, edge counts, device/pin/net counts, number of labels, and summary statistics for each analog circuit.

2. **Full SRAM statistics table**  
   Include node counts, edge counts, and separate summaries for `Cg`, `Cc`, and `Reff`.

3. **Optional feature-dimension table**  
   A small table summarizing the dimensionality of device, net, and pin features for each released graph format.

---

## B.4 Data families and task targets

The benchmark supports both **node-level** and **edge-level** prediction.

### Node-level target

- **Ground capacitance (`Cg`)** assigned to net nodes.

### Edge-level targets

- **Coupling capacitance (`Cc`)** between corresponding net pairs,
- **Effective resistance (`Reff`)** between pin pairs that belong to the same resistive network.

For some subsets, not every target is present in the same way. When preparing the final public repository, it is helpful to state explicitly for each dataset:

- which targets are available,
- whether the task is intended for classification, regression, or both,
- whether the benchmark uses all labels or a filtered subset.

**Suggested insertion point:**
- add a small matrix here mapping dataset name → available targets → task modes.

---

## B.5 Dataset sourcing and physical validity

All ParasGB data should be described as coming from **real design flows** and **post-layout parasitic extraction**, rather than from purely synthetic graphs. The benchmark is valuable precisely because it sits closer to industrial signoff conditions than prior schematic-only circuit benchmarks.

A concise repository-facing way to state this is:

> The graphs are derived from schematic/layout design data and commercial parasitic extraction flows. The released labels represent post-layout parasitic quantities used to benchmark graph learning models under realistic RC prediction conditions.

If you want, you can place here:

- a small flowchart thumbnail,
- one representative extraction-to-graph conversion image,
- a release note explaining which raw internal artifacts are not shared for confidentiality reasons.

---

## B.6 Feature description

The main paper describes a hierarchical feature design at the **device**, **net**, and **pin** levels.

### Device-level features

Typical examples include:

- device type indicators,
- width `W`,
- length `L`,
- multiplier,
- number of fingers,
- process- or role-related identifiers.

### Net-level features

Typical examples include:

- power / ground / signal / I/O flags,
- connectivity statistics,
- aggregated geometric descriptors,
- counts of attached device terminals.

### Pin-level features

Typical examples include:

- drain / gate / source / bulk indicators,
- capacitor / resistor terminal indicators,
- local terminal-role encodings.

### Suggested addition

If you have the final released field names, add a complete feature table here or attach it as a CSV / PDF in `tables/` and link it below.

**Files to link here:**
- `tables/analog/table_B1_analog_statistics.csv`
- `tables/sram/table_B2_sram_statistics.csv`
- `tables/analog/table_B3_analog_feature_definitions.csv`
- `tables/sram/table_B4_sram_feature_definitions.csv`

---

## B.7 Label-distribution notes

One of the most important observations in ParasGB is that the parasitic labels are not cleanly balanced. The distributions are typically:

- highly skewed,
- long-tailed,
- occasionally multi-modal,
- very different across circuit family and scale.

This matters because the benchmark is not only a test of representational power; it is also a test of how robust a model is under difficult target distributions.

### What to show here

This section is a good place to add:

- full histogram panels for each split,
- log-domain and raw-domain views side by side,
- per-target quantile summaries,
- rare-bin counts for the classification setting.

---

## B.8 Release checklist for this appendix

Before publishing, make sure this appendix page links to:

- [ ] analog statistics table,
- [ ] SRAM statistics table,
- [ ] analog label-distribution figures,
- [ ] SRAM label-distribution figures,
- [ ] feature-definition table,
- [ ] optional split manifest.

---

## B.9 Placeholder links

Replace the placeholders below once the files are added:

- Analog statistics table: `../tables/analog/table_B1_analog_statistics.csv`
- SRAM statistics table: `../tables/sram/table_B2_sram_statistics.csv`
- Analog label histograms: `../figures/appendix/fig_B1_analog_label_histograms.png`
- SRAM label histograms: `../figures/appendix/fig_B2_sram_label_histograms.png`
- Feature visualization: `../figures/appendix/fig_B3_feature_space_visualization.png`
