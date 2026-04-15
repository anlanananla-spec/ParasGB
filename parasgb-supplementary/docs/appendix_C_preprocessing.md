# Appendix C — Preprocessing, Normalization, Evaluation, and Experimental Notes

## Purpose of this document

This page collects the preprocessing and experimental details that support reproducibility of the ParasGB benchmark. It is the right place to explain how raw values become model-ready targets, how splits are handled, and how evaluation is standardized across methods.

---

## C.1 Why preprocessing is necessary

Circuit parasitic quantities can vary across multiple orders of magnitude. In addition, graph attributes such as coordinates, device sizes, and parasitic labels may live on very different numeric scales. Directly training on raw values can therefore lead to unstable optimization, poor gradient behavior, and inconsistent evaluation.

For that reason, ParasGB adopts a unified preprocessing workflow before training or benchmarking.

---

## C.2 Target normalization

### Continuous targets

For regression tasks, the benchmark uses normalized continuous values. The main paper indicates that a logarithmic transform is applied before mapping values into a common numeric range.

A concise description you can keep here is:

1. start from the raw parasitic value,
2. apply a log-domain transform,
3. map the transformed value into a normalized interval,
4. store the normalized value as the regression target.

### What to add here later

Please insert your exact released formula once finalized. For example:

```text
x_raw -> x_log = f(x_raw) -> x_norm = g(x_log)
```

You may also want to document:

- clipping rules,
- epsilon constants,
- treatment of zero or near-zero capacitance values,
- whether each target type uses a different normalization range.

---

## C.3 Classification label generation

For classification tasks, normalized continuous labels are discretized into bins.

This section should clearly answer:

- how many classes are used,
- whether bins are uniform, quantile-based, or manually defined,
- whether boundaries are shared across datasets or target-specific,
- whether outliers are clipped before discretization.

### Recommended text to keep

> Classification labels are derived from normalized parasitic values by discretizing the continuous targets into predefined magnitude intervals. The resulting bins are used to benchmark early-stage screening and coarse-grained parasitic estimation.

### Suggested insertion point

Add a compact table of bin boundaries here when ready.

---

## C.4 Data splits and benchmarking protocol

The repository version of this appendix should explain how each model is trained and evaluated under a standardized protocol.

Key questions to answer here:

- Which datasets are used for training, validation, and test?
- Are analog experiments cross-scale, cross-design, or leave-one-design-out?
- Are SRAM experiments trained on small graphs and tested on larger ones?
- Are all reported results produced under the same split files?

### Recommended additions

1. A small split summary table.
2. A manifest file in `metadata/` that records split membership.
3. A note clarifying whether split files are fixed across all released baselines.

**Placeholder file:**
- `../metadata/splits_manifest.md`

---

## C.5 Evaluator and reported metrics

ParasGB evaluates both classification and regression settings.

### Classification metrics

Typical metrics include:

- Accuracy,
- F1-score.

### Regression metrics

Typical metrics include:

- MAE,
- R².

You may want to add here:

- whether macro-F1 or weighted-F1 is used,
- whether regression metrics are computed in normalized space or mapped-back physical space,
- whether results are averaged over multiple seeds.

---

## C.6 Loader and implementation notes

The paper mentions a PyTorch Geometric style benchmark interface with task-specific loaders. This appendix is a good place to briefly document:

- which loader is used for node tasks,
- which loader is used for edge tasks,
- whether graph caching is enabled,
- whether preprocessing artifacts are saved after first use.

### Suggested text

> The benchmark implementation is designed to integrate with PyTorch Geometric. Node-level tasks and edge-level tasks may use different loading strategies to support scalability on large circuit graphs. Preprocessed graph objects can be cached to improve reproducibility and reduce repeated overhead.

---

## C.7 Hyperparameters and training settings

This section can hold the detailed training configurations that are too long for the main paper.

### Recommended contents

- optimizer,
- learning rate,
- weight decay,
- batch size,
- number of epochs,
- early stopping rule,
- number of runs / seeds,
- hardware platform,
- memory constraints,
- task-specific sampling parameters.

### Suggested tables to add

- `table_C1_common_hyperparameters.csv`
- `table_C2_model_specific_hyperparameters.csv`
- `table_C3_hardware_configuration.csv`

---

## C.8 Notes on imbalance and robustness

The benchmark is intentionally difficult because the label distributions are highly imbalanced. It is useful to document here whether any of the following are part of the standardized benchmark or only part of optional baselines:

- resampling,
- reweighting,
- focal losses,
- balanced MSE / distribution-aware regression losses,
- class-frequency smoothing,
- edge subsampling.

If the repository release includes optional baseline-specific tricks, it is best to separate them from the default protocol so readers understand what is benchmark-standard and what is model-specific.

---

## C.9 Reproducibility checklist

Before release, verify that this appendix or linked files specify:

- [ ] normalization formula,
- [ ] class boundaries,
- [ ] fixed split definition,
- [ ] evaluator metrics,
- [ ] training seeds,
- [ ] hardware notes,
- [ ] caching / preprocessing behavior.

---

## C.10 Placeholder links

Replace the placeholders below once files are added:

- Normalization figure: `../figures/appendix/fig_C1_normalization_pipeline.png`
- Discretization figure: `../figures/appendix/fig_C2_class_binning.png`
- Common hyperparameters: `../tables/analog/table_C1_common_hyperparameters.csv`
- Split manifest: `../metadata/splits_manifest.md`
