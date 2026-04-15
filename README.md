# ParasGB Supplementary Materials

## Repository Structure

This repository is organized as a compact supplementary material collection for the paper *ParasGB: A Graph Benchmark Suite for Parasitic Estimation on AMS Circuits*. It is not intended to function as a standalone code repository. Instead, it gathers the visual and tabular materials that support the benchmark and make the paper easier to inspect in detail.

The repository mainly stores three types of content. First, it includes figures that illustrate the benchmark construction process, label distributions, feature-space visualizations, and extended experimental observations. Second, it includes tables that summarize dataset statistics, task settings, and benchmark results that are useful for close comparison but are easier to browse separately from the paper. Third, it may include a small amount of lightweight descriptive material when necessary to clarify file meanings, naming conventions, or the relation between a stored item and the corresponding part of the benchmark.

Taken together, the materials collected here are intended to support a more detailed reading of ParasGB from the perspectives of dataset composition, task design, preprocessing, graph construction, label generation, empirical evaluation, and future benchmark development. The repository therefore serves as a structured companion resource for readers who would like to inspect the benchmark more closely than the main paper alone allows.

## 1. Benchmark Usage and Evaluation Protocol

The first group of materials documents how the benchmark is intended to be used in practice. ParasGB is designed to be compatible with the PyTorch Geometric ecosystem and provides a standardized evaluation protocol so that different models can be compared under a common interface. The benchmark supports both analog circuits and SRAM circuits, and it covers node-level as well as edge-level prediction settings. It also supports both classification and regression tasks, with accuracy and F1 used for classification, and MAE and R² used for regression.

The purpose of this part is to make the benchmark reproducible and easy to adopt. Readers can use it to understand how dataset objects are loaded, how task types are defined, how training and evaluation are expected to be performed, and how the benchmark exposes split indices and metadata. For large-scale SRAM graphs, this part is especially important because efficient loaders and sampling strategies are necessary for stable training on graphs with very large numbers of nodes and edges.

In this repository, the corresponding materials mainly help explain how ParasGB is accessed, what evaluation conventions are followed, and how researchers should interpret the reported results under a unified benchmark setting.

## 2. Dataset Composition and Label Content

The second group of materials focuses on the composition of the benchmark itself. ParasGB contains both analog and SRAM circuit data and is deliberately constructed to span very different graph scales. The analog portion covers representative AMS modules such as LVDS, BGR, OP, and LDO, while the SRAM portion includes designs ranging from relatively small instances to extremely large industrial-scale arrays.

This part of the repository is meant to show the diversity, scale, and physical grounding of the dataset more clearly. The corresponding tables and figures provide expanded statistics on numbers of devices, pins, nets, nodes, and edges, as well as the ranges and distributions of benchmark targets. They also clarify which prediction targets are attached to nodes and which are attached to edges.

Three major parasitic targets are represented in ParasGB. Ground capacitance (`Cg`) is treated as a node-level target. Coupling capacitance (`Cc`) and effective resistance (`Reff`) are treated as edge-level targets. These labels are drawn from post-layout parasitic information rather than schematic-level approximations, which is a key reason the benchmark is relevant to realistic AMS and memory design flows.

This part also includes materials that explain feature definitions and circuit sources in greater detail. In particular, it helps readers understand how SRAM and analog circuits differ in scale, what kinds of modules are included, and how the benchmark combines small but physically delicate analog graphs with very large and computationally demanding SRAM graphs.

## 3. Preprocessing, Baselines, and Experimental Settings

The third group of materials explains how the benchmark is prepared for model training and how the experiments are configured. Since parasitic values span multiple orders of magnitude, preprocessing is a central part of the benchmark design. The associated materials therefore explain value filtering, normalization, and the discretization process used in classification settings. These steps are important because they affect both optimization stability and task difficulty.

This section also introduces the benchmark baselines in a more systematic way. The benchmark compares classical message-passing GNNs, graph transformer variants, and circuit-specific models. The goal is not only to report raw performance numbers, but also to reveal what kinds of graph learning architectures remain effective under strong label imbalance, heterogeneous graph structure, and cross-scale generalization pressure.

The materials here additionally summarize the concrete experimental settings used for analog circuits and SRAM circuits. This includes hidden dimensions, learning rates, dropout, batch sizes, sampling strategies, and model-specific adjustments. Since analog tasks and ultra-large SRAM tasks place very different demands on memory and model capacity, these details are necessary to interpret the fairness and reproducibility of the benchmark results. Hardware and software configuration summaries are also included here to make the evaluation environment transparent.

## 4. Extended Regression Results

The fourth group of materials presents the extended regression results that complement the main classification-oriented discussion in the paper. While classification results are useful for showing whether a model can distinguish coarse parasitic magnitude levels, regression results are essential for understanding whether a model can recover the actual numerical values of the parasitic targets.

This part therefore provides a fuller view of the benchmark difficulty. It covers node-level regression for ground capacitance and edge-level regression for coupling capacitance and effective resistance. The corresponding tables allow readers to compare MAE and R² across different datasets and model families, and to see more clearly how numerical prediction difficulty varies across target type and circuit scale.

These results are particularly useful because they reveal that success in classification does not automatically translate into strong regression accuracy. In large SRAM designs, regression can become much harder because of scale, sparsity, and long-tailed label behavior. In some analog settings, regression is comparatively more stable, which highlights the layered nature of physical modeling difficulty across the benchmark.

## 5. Current Scope and Limitations

The fifth group of materials explains the present limitations of the benchmark. This part is important because ParasGB is meant to be a realistic and transparent benchmark, not an overly simplified or overly generalized claim about all circuit parasitic prediction settings.

The main limitations concern coverage, precision, technology transfer, and physical modeling depth. The current benchmark centers on SRAM and several representative analog modules, which are highly meaningful but do not exhaust the full range of industrial circuit types. Regression accuracy also remains challenging, especially in long-tailed settings where extreme parasitic values are difficult to predict reliably. In addition, the dataset is not yet broad enough to fully validate cross-node or cross-foundry generalization. Finally, although the graph representation captures topology and some geometric or device-related information, deeper physical interactions such as richer 3D coupling effects are not yet modeled exhaustively.

This part of the repository is therefore intended to frame ParasGB honestly: it is a strong and useful benchmark, but it is also an evolving benchmark whose current scope should be understood with appropriate precision.

## 6. Future Development Directions

The sixth group of materials describes the most important directions for future extension of the benchmark and of graph learning methods for parasitic estimation. These materials are included to show how ParasGB can grow beyond its current release and what open technical opportunities remain.

Several directions are especially important. One is the exploration of graph foundation model pre-training on large-scale circuit data so that transferable physical priors can be learned before task-specific fine-tuning. Another is the improvement of spatial geometry perception, especially by integrating more explicit 3D layout and metal-layer information into graph-based modeling. A third direction is the extension of the benchmark to a broader circuit family coverage, including additional analog, RF, interface, digital, and mixed-signal scenarios across multiple process nodes. A further long-term direction is to move from purely offline benchmarking toward design-loop integration, where predictive models can provide earlier and more practical guidance during layout optimization.

This part is useful because it clarifies that ParasGB is not only a static dataset release, but also a starting point for a broader benchmark and modeling agenda in learning-based EDA.

## 7. Analog Circuit Graph Construction

The seventh group of materials explains how analog circuit topologies are converted into graph representations. This part complements the main graph construction description by focusing on the analog side in a more explicit way.

The central idea is to represent each circuit as a heterogeneous graph containing device nodes, pin nodes, and net nodes. Topological edges encode the structural connectivity extracted from the schematic, especially device-to-pin and pin-to-net relations. The parasitic quantities, by contrast, are not treated as input structure. They are used as prediction targets attached either to nodes or to edges after extraction from post-layout information.

This separation is important for the benchmark definition. It means the model is asked to infer parasitic behavior from circuit topology and attributes, rather than being directly given the parasitic network as part of the input graph. The visual materials stored for this part of the repository help readers understand how the original circuit description, the extracted RC information, the simplified representation, and the final graph view are connected.

## 8. Effective Resistance Label Generation

The eighth group of materials documents the algorithmic side of effective resistance label generation. Effective resistance is one of the core prediction targets in ParasGB, and it is physically meaningful but computationally nontrivial to obtain at scale.

The materials in this part explain the matrix-based procedure used to calculate effective resistance efficiently. Instead of relying on direct path-based simulation for every queried pair, the benchmark constructs an admittance matrix from the resistive network, removes a reference node to obtain an invertible system, and then computes effective resistances through matrix factorization. The use of the inverse of the Cholesky factor makes large-scale label generation significantly more practical.

This part is especially important because it shows how ParasGB turns extracted physical circuit information into usable edge-level supervision at benchmark scale. It also clarifies that the benchmark is grounded in a physically motivated label-generation pipeline rather than in arbitrary graph annotations.
