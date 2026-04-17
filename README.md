# ParasGB

## Table of Contents

- [1. User Guide](#1-user-guide)
- [2. Dataset Details](#2-dataset-details)
- [3. Experiment Details](#3-experiment-details)
- [4. Additional Regression Task Results](#4-additional-regression-task-results)
- [5. Analog Topology-to-Graph](#5-analog-topology-to-graph)
- [6. Algorithms](#6-algorithms)
- [7. Limitations](#7-limitations)
- [8. Future Directions](#8-future-directions)

---

## 1. User Guide

### 1.1 Standardized Evaluation Protocol

ParasGB is deeply integrated with **PyTorch Geometric (PyG)** so that researchers can complete the following steps with only a small amount of code:

- dataset download
- graph data preprocessing
- task-level data loading
- standardized evaluation

To support ultra-large circuit graphs, the toolkit provides:

- `NeighborLoader`: for node-level tasks
- `LinkNeighborLoader`: for edge-level tasks

Both loaders support caching preprocessed graph data on first use, which reduces repeated computation, improves reproducibility, and avoids experimental discrepancies caused by differences in preprocessing logic.

In addition, ParasGB provides a unified `Evaluator` module that follows the OGB style and automatically reports standardized metrics:

- classification tasks: Accuracy / F1
- regression tasks: MAE / R²

The dataset object also exposes metadata such as:

- train/test splits
- number of nodes and edges
- label distributions

This makes it easier to compare the real performance of different models across different tasks in a transparent way.

### 1.2 ParasGB Usage

The core goal of ParasGB is to **lower the barrier to parasitic-parameter learning research**. Its usage style is intentionally close to PyG. Researchers only need to specify:

- dataset name
- task level (`node` / `edge`)
- task type (`classification` / `regression`)

The system will then automatically complete raw file download and feature preprocessing.

For SRAM graphs with tens of millions of nodes, the toolkit provides subgraph sampling for limited-memory settings to ensure both training efficiency and stability.

### Task Name Reference

#### SRAM

- `cg_regr`: node-level ground capacitance regression
- `cg_class`: node-level ground capacitance classification
- `cc_regr`: edge-level coupling capacitance regression
- `cc_class`: edge-level coupling capacitance classification
- `r_class`: edge-level effective resistance classification

#### Analog

- `cg_regr`: node-level ground capacitance regression
- `cg_class`: node-level ground capacitance classification
- `r_regr`: edge-level effective resistance regression
- `r_class`: edge-level effective resistance classification

### Minimal Usage Example

```python
from parasgb import RCDataset, Evaluator

# 1. Load dataset with predefined train/test splits
# dataset_name: 'sram' or 'analog'
# task_level: 'node' or 'edge'
# task_type: 'regression' or 'classification'
dataset = RCDataset(
    dataset_name='sram',
    root='data/',
    task_level='node',
    task_type='regression'
)

# 2. Create DataLoader
train_loader = dataset.get_dataloader(
    split='train',
    batch_size=32,
    shuffle=True
)

# 3. Define model and optimizer
model = MyModel()
criterion = Loss()
optimizer = Optimizer(model.parameters())

# 4. Train
for epoch in range(E):
    for batch in train_loader:
        y_pred = model(batch.node_attr, batch.edge_index)
        loss = criterion(y_pred.squeeze(), batch.y[:, 0])
        loss.backward()
        optimizer.step()
        optimizer.zero_grad()

# 5. Create evaluator
# Example: SRAM node-level ground capacitance regression
evaluator = Evaluator(
    dataset_name='sram',
    task='cg_regr'
)

# 6. Evaluate
metrics = evaluator.evaluate(y_pred, y_true)
print(f"MAE = {metrics['mae']:.4f}")
```

---

## 2. Dataset Details

### 2.1 SRAM

The SRAM subset uses **statistically aggregated features**, focusing on global topology and device distribution rather than preserving overly fine-grained local device details. This design controls feature dimensionality in ultra-large-graph scenarios while still retaining physically meaningful statistical information.

The main characteristics of the SRAM data are:

- extremely large graph size
- highly repetitive topology
- dense routing
- high coupling-capacitance density
- well suited for evaluating **scalability** and **computational efficiency**

The main subsets include:

- `ssram`: a baseline SRAM design used to verify whether a model can capture regular topology
- `digtime`: a digital timing-related logic module
- `timing ctrl`: an internal timing-control module in memory, with more complex topology
- `sandwich`: a stacked high-performance memory architecture with very high coupling density
- `ultra8t`: an 8T SRAM optimized for subthreshold low-power operation
- `array 128 32 8t`: one of the largest arrays in the dataset, used for extreme stress testing

The node features of SRAM circuit graphs are defined as follows:

### Definition of SRAM Circuit Graph Node Features

#### Device

| Type | Feature | Definition | Index |
| --- | --- | --- | --- |
| Device | $M_{mos}$ | Multiplier of transistors | 0 |
| Device | $L$ | Length of the transistor | 1 |
| Device | $W$ | Width of the transistor | 2 |
| Device | $M_{res}$ | Multiplier of connected resistors | 3 |
| Device | $L_{res}$ | Length of resistor | 4 |
| Device | $W_{res}$ | Width of resistor | 5 |
| Device | $M_{cap}$ | Multiplier of connected capacitor | 6 |
| Device | $L_{r}$ | Length of capacitor | 7 |
| Device | $N_{r}$ | Number of capacitor fingers | 8 |
| Device | $N_{p}$ | Number of ports in the device instance | 9 |
| Device | $T$ | Type code of the device instance | 10 |

#### Net

| Type | Feature | Definition | Index |
| --- | --- | --- | --- |
| Net | $N_{mos}$ | Number of connected transistors | 0 |
| Net | $N_{g}$ | Number of connected gate terminals | 1 |
| Net | $N_{sd}$ | Number of connected source/drain terminals | 2 |
| Net | $N_{b}$ | Number of connected base terminals | 3 |
| Net | $W_{tot}$ | Total width of connected transistor | 4 |
| Net | $L_{tot}$ | Total length of connected transistor | 5 |
| Net | $N_{cap}$ | Number of connected capacitors | 6 |
| Net | $Lr_{tot}$ | Total length of connected capacitors | 7 |
| Net | $Nr_{tot}$ | Total number of connected capacitor fingers | 8 |
| Net | $N_{res}$ | Number of connected resistors | 9 |
| Net | $W_{tot,res}$ | Total width of connected resistors | 10 |
| Net | $L_{tot,res}$ | Total length of connected resistors | 11 |
| Net | $N_{port}$ | Number of connected ports | 12 |

#### Pin

| Type | Feature | Definition | Index |
| --- | --- | --- | --- |
| Pin | -- | Pin types (G/D/S/B for MOS) | 0 |

### 2.2 Analog

Unlike the SRAM dataset, the analog dataset contains smaller circuits but much more detailed device-level physical descriptions. Key parameters, such as device channel width `W`, length `L`, and the distance from the source/drain region to the isolation edge (the LDE effect), are included in the node-feature system. Circuit parasitic parameters are extracted using commercial PEX tools. Because analog circuits are highly sensitive to noise, even very small parasitic prediction errors can cause simulation results to deviate from expectations.

The dataset lists the sources and functional summaries of 20 analog cases, including:

- `ID 1`: an LVDS circuit that converts a low-frequency reference (5–27 MHz) into a high-frequency clock (100–700 MHz) with minimal phase noise for precise timing applications (Leung & Mok, 2003b).
- `ID 2`: an operational amplifier circuit that exploits quasi-linear temperature characteristics to generate a stable 0.4 V reference from an ultra-low 0.56 V supply while consuming only 4.8 μA current (Wang & Ye, 2006).
- `ID 3`: a bandgap reference circuit that uses a self-cascode composite transistor and a single resistor to generate a stable reference voltage close to the silicon bandgap, achieving a low temperature coefficient of 25.3 ppm/°C with only 25 μA current (Colombo et al., 2012).
- `ID 4`: a bandgap reference circuit that uses resistor subdivision and a resistorless method to generate a highly stable 910.88 mV reference voltage with an ultra-low temperature coefficient of 12.99 ppm/°C (Koh & Lee, 2014).
- `ID 5`: an LDO circuit that balances the temperature characteristics of N/P-type MOSFETs to generate a stable reference voltage for the LDO regulator, achieving a temperature coefficient of 36.9 ppm/°C with a low supply current of 9.7 μA (Leung & Mok, 2003a).
- `ID 6`: an LDO circuit that provides a stable output from a 1.8–4.5 V supply with fast transient response and minimal compensation capacitance (7 pF), supporting up to 100 mA load current with a 0.2 V dropout (Ho & Mok, 2010a).
- `ID 7`: an LDO circuit that balances the temperature characteristics of N/P-type MOSFETs to generate a stable reference voltage for the LDO regulator, achieving a temperature coefficient of 36.9 ppm/°C with a low supply current of 9.7 μA (Leung & Mok, 2003a).
- `ID 8`: an LDO circuit that uses an adaptive compensation buffer (ACB) to dynamically switch between pass transistors, enabling stable operation over a wide load range (0 to 30 mA) without external capacitors while maintaining a low quiescent current of 6 μA (Tan et al., 2025).
- `ID 9`: an LDO circuit that uses a three-loop architecture to achieve ultra-fast transient response (1.15 ns) while maintaining a clean power supply, with full-spectrum power-supply rejection (PSR > −12 dB up to 20 GHz) and only 50 μA quiescent current (Lu et al., 2015).
- `ID 10`: an operational amplifier circuit that provides a flexible, step-by-step method for balancing noise performance and power consumption, offering greater design control than previous methods and validated by multi-condition SPICE simulations (Mahattanakul & Chutichatuporn, 2005).
- `ID 11`: an LDO circuit that uses two parallel active-feedback paths to create two pole-zero pairs, providing superior stability and transient response compared with single-path methods, while supporting a 100 mA load with only 14 μA quiescent current (Li et al., 2020a).
- `ID 12`: an LDO circuit that uses a high-gain three-stage error amplifier to maintain accurate regulation even at ultra-low voltage (0.5 V supply) with an unsaturated pass transistor, achieving a current density of 11.4 A/mm² and a low-frequency PSR of −62 dB (Kim & Cho, 2023).
- `ID 13`: an operational amplifier circuit that replaces traditional Miller compensation with an active structure, eliminates the right-half-plane (RHP) zero, and introduces a left-half-plane (LHP) zero to cancel the first non-dominant pole, increasing the unity-gain frequency by 9.4× while significantly reducing compensation capacitance (Tan & Zhou, 2011).
- `ID 14`: a bandgap reference circuit that uses a combination of four MOSFETs, two lateral PNP transistors, and a well resistor to generate a stable 16 μA output current with a temperature coefficient of 105 ppm/°C, without requiring an external bandgap reference or trimming-process compensation (Osipov & Paul, 2017).
- `ID 15`: an LDO circuit that uses damping-zero compensation and a slew-rate enhancement circuit to achieve both stability and fast transients, with only 1.5 pF on-chip capacitance while supporting a 100 mA load and 200 mV dropout (Ho & Mok, 2010b).
- `ID 16`: an LDO circuit that uses a WCF circuit to maintain fast transient response and stable voltage regulation over a very wide range of load current (up to 100 mA) and load capacitance (470 pF to 10 nF), while consuming only 14.4 μA power (Wang et al., 2016).
- `ID 17`: an LDO circuit that uses a nested adaptive FVF structure to achieve ultra-fast transient response (handling load steps from 1 μA to 20 mA in only 10 ps), while significantly improving PSR (−58.52 dB at 1 MHz) and line regulation (Li et al., 2020b).
- `ID 18`: an LDO circuit that provides stable output with high DC gain (101 dB) and an accurate bandgap reference, supporting large-current loads (up to 450 mA) with only 0.5 V dropout while maintaining solid power-supply rejection (54.5 dB at 100 Hz) (Martínez-García et al., 2013).
- `ID 19`: a bandgap reference circuit that uses size-dependent effects to cancel process-induced threshold-voltage variation, achieving ultra-low power consumption of 192 pW and highly stable performance (0.53% process variation) without post-fabrication trimming (Ji et al., 2019).
- `ID 20`: a bandgap reference circuit that uses an ultra-low-power architecture to generate a stable reference voltage, with most of the 5 μA current dedicated to output, achieving a temperature coefficient below 10 ppm/°C from a 1 V supply without requiring a large-area operational amplifier (Edward, 2009).

### 2.3 Dataset Labels

Label distribution plots:

- **Analog Reff**: effective resistance distributions for 20 analog circuits

<table>
  <tr>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case1_RC_edge_normalized.png" width="180"><br>
      <sub>(a) Case 1</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case5_RC_edge_normalized.png" width="180"><br>
      <sub>(b) Case 2</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case7_RC_edge_normalized.png" width="180"><br>
      <sub>(c) Case 3</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case10_RC_edge_normalized.png" width="180"><br>
      <sub>(d) Case 4</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case11_RC_edge_normalized.png" width="180"><br>
      <sub>(e) Case 5</sub>
    </td>
  </tr>

  <tr>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case15_RC_edge_normalized.png" width="180"><br>
      <sub>(f) Case 6</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case17_RC_edge_normalized.png" width="180"><br>
      <sub>(g) Case 7</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case23_RC_edge_normalized.png" width="180"><br>
      <sub>(h) Case 8</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case29_RC_edge_normalized.png" width="180"><br>
      <sub>(i) Case 9</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case39_RC_edge_normalized.png" width="180"><br>
      <sub>(j) Case 10</sub>
    </td>
  </tr>

  <tr>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case42_RC_edge_normalized.png" width="180"><br>
      <sub>(k) Case 11</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case44_RC_edge_normalized.png" width="180"><br>
      <sub>(l) Case 12</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case45_RC_edge_normalized.png" width="180"><br>
      <sub>(m) Case 13</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case55_RC_edge_normalized.png" width="180"><br>
      <sub>(n) Case 14</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case58_RC_edge_normalized.png" width="180"><br>
      <sub>(o) Case 15</sub>
    </td>
  </tr>

  <tr>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case71_RC_edge_normalized.png" width="180"><br>
      <sub>(p) Case 16</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case72_RC_edge_normalized.png" width="180"><br>
      <sub>(q) Case 17</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case74_RC_edge_normalized.png" width="180"><br>
      <sub>(r) Case 18</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case75_RC_edge_normalized.png" width="180"><br>
      <sub>(s) Case 19</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case78_RC_edge_normalized.png" width="180"><br>
      <sub>(t) Case 20</sub>
    </td>
  </tr>
</table>

- **Analog Cg**: ground-capacitance distributions for 20 analog circuits

<h3 align="center">Analog RC Node Label Distributions</h3>

<table>
  <tr>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case1_RC_normalized.png" width="180"><br>
      <sub>(a) Case 1</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case5_RC_normalized.png" width="180"><br>
      <sub>(b) Case 2</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case7_RC_normalized.png" width="180"><br>
      <sub>(c) Case 3</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case10_RC_normalized.png" width="180"><br>
      <sub>(d) Case 4</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case11_RC_normalized.png" width="180"><br>
      <sub>(e) Case 5</sub>
    </td>
  </tr>

  <tr>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case15_RC_normalized.png" width="180"><br>
      <sub>(f) Case 6</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case17_RC_normalized.png" width="180"><br>
      <sub>(g) Case 7</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case23_RC_normalized.png" width="180"><br>
      <sub>(h) Case 8</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case29_RC_normalized.png" width="180"><br>
      <sub>(i) Case 9</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case39_RC_normalized.png" width="180"><br>
      <sub>(j) Case 10</sub>
    </td>
  </tr>

  <tr>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case42_RC_normalized.png" width="180"><br>
      <sub>(k) Case 11</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case44_RC_normalized.png" width="180"><br>
      <sub>(l) Case 12</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case45_RC_normalized.png" width="180"><br>
      <sub>(m) Case 13</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case55_RC_normalized.png" width="180"><br>
      <sub>(n) Case 14</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case58_RC_normalized.png" width="180"><br>
      <sub>(o) Case 15</sub>
    </td>
  </tr>

  <tr>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case71_RC_normalized.png" width="180"><br>
      <sub>(p) Case 16</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case72_RC_normalized.png" width="180"><br>
      <sub>(q) Case 17</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case74_RC_normalized.png" width="180"><br>
      <sub>(r) Case 18</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case75_RC_normalized.png" width="180"><br>
      <sub>(s) Case 19</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case78_RC_normalized.png" width="180"><br>
      <sub>(t) Case 20</sub>
    </td>
  </tr>
</table>

- **SRAM Cc**: coupling-capacitance distributions for 6 SRAM circuits

<table>
  <tr>
    <td align="center">
      <img src="IMGS/sram/edge_each_label/array_128_32_8t_normalized.png" width="160"><br>
      <sub>(a) Array_128_32_8t</sub>
    </td>
    <td align="center">
      <img src="IMGS/sram/edge_each_label/digtime_normalized.png" width="160"><br>
      <sub>(b) Digtime</sub>
    </td>
    <td align="center">
      <img src="IMGS/sram/edge_each_label/sandwich_normalized.png" width="160"><br>
      <sub>(c) Sandwich</sub>
    </td>
    <td align="center">
      <img src="IMGS/sram/edge_each_label/ssram_normalized.png" width="160"><br>
      <sub>(d) SSRAM</sub>
    </td>
    <td align="center">
      <img src="IMGS/sram/edge_each_label/timing_ctrl_normalized.png" width="160"><br>
      <sub>(e) Timing_Ctrl</sub>
    </td>
    <td align="center">
      <img src="IMGS/sram/edge_each_label/ultra8t_normalized.png" width="160"><br>
      <sub>(f) Ultra8t</sub>
    </td>
  </tr>
</table>

- **SRAM Reff**: effective-resistance distributions for 6 SRAM circuits

<table>
  <tr>
    <td align="center">
      <img src="IMGS/sram/edge_r_each_label/array_128_32_8t_normalized.png" width="160"><br>
      <sub>(a) Array_128_32_8t</sub>
    </td>
    <td align="center">
      <img src="IMGS/sram/edge_r_each_label/digtime_normalized.png" width="160"><br>
      <sub>(b) Digtime</sub>
    </td>
    <td align="center">
      <img src="IMGS/sram/edge_r_each_label/sandwich_normalized.png" width="160"><br>
      <sub>(c) Sandwich</sub>
    </td>
    <td align="center">
      <img src="IMGS/sram/edge_r_each_label/ssram_normalized.png" width="160"><br>
      <sub>(d) SSRAM</sub>
    </td>
    <td align="center">
      <img src="IMGS/sram/edge_r_each_label/timing_ctrl_normalized.png" width="160"><br>
      <sub>(e) Timing_Ctrl</sub>
    </td>
    <td align="center">
      <img src="IMGS/sram/edge_r_each_label/ultra8t_normalized.png" width="160"><br>
      <sub>(f) Ultra8t</sub>
    </td>
  </tr>
</table>

- **SRAM Cg**: ground-capacitance distributions for 6 SRAM circuits

<table>
  <tr>
    <td align="center">
      <img src="IMGS/sram/node_each_label/array_128_32_8t_normalized.png" width="160"><br>
      <sub>(a) Array_128_32_8t</sub>
    </td>
    <td align="center">
      <img src="IMGS/sram/node_each_label/digtime_normalized.png" width="160"><br>
      <sub>(b) Digtime</sub>
    </td>
    <td align="center">
      <img src="IMGS/sram/node_each_label/sandwich_normalized.png" width="160"><br>
      <sub>(c) Sandwich</sub>
    </td>
    <td align="center">
      <img src="IMGS/sram/node_each_label/ssram_normalized.png" width="160"><br>
      <sub>(d) SSRAM</sub>
    </td>
    <td align="center">
      <img src="IMGS/sram/node_each_label/timing_ctrl_normalized.png" width="160"><br>
      <sub>(e) Timing_Ctrl</sub>
    </td>
    <td align="center">
      <img src="IMGS/sram/node_each_label/ultra8t_normalized.png" width="160"><br>
      <sub>(f) Ultra8t</sub>
    </td>
  </tr>
</table>

Main observations:

- ground-capacitance labels generally show a **clear long-tailed distribution**
- SRAM labels span a wider range, making regression more difficult
- effective-resistance labels in analog circuits are sparser than capacitance labels and contain more outliers
- the number of coupling-capacitance edges in SRAM is much larger than the number of ground-capacitance node labels, making it a key scenario for evaluating non-local modeling ability

---

## 3. Experiment Details

### 3.1 Data Preprocessing

#### SRAM Task Filtering Rules

- `Cg` node task: keep `(1e-21, 1e-15) F`
- `Cc` edge task: keep `(1e-21, 1e-15) F`
- `Reff` edge task: filter zero values and apply global `1% - 99%` quantile normalization

#### Analog Task Filtering Rules

- `Cg` node task: keep `(0, 8e-13) F`
- `Reff` edge task: keep `[0, 700] Ω`

All classification tasks use **five equal-width bins**.

### 3.2 Baseline Details

This study selects a series of representative graph-learning models for comparative experiments to validate the challenge level of the ParasGB benchmark. The models are grouped as follows.

#### 1) Classical Message-Passing GNNs

These models are mainstream methods in graph learning. Their core idea is to learn local topological structure in circuits through feature propagation and aggregation among neighboring nodes. They offer efficient computation and relatively low memory consumption, making them some of the most widely used baselines in circuit-related tasks.

- `GCN` (Kipf & Welling, 2017): a classic graph convolutional network that aggregates neighbor features through Laplacian smoothing.
- `GAT` (Velicković et al., 2018): an attention-based graph neural network that adaptively weights neighboring nodes during message passing.
- `GraphSAGE` (Hamilton et al., 2017): an inductive graph representation model that learns node embeddings through neighborhood sampling and aggregation.
- `PNA` (Corso et al., 2020): a message-passing architecture that combines multiple aggregators and degree-scalers to improve expressive power.

#### 2) Graph Transformers

Unlike traditional models that only capture local neighborhoods, these models introduce global attention mechanisms and can model global topological dependencies across the entire circuit. For large circuit networks such as SRAM, which exhibit repetitive topology and long-range coupling effects, such models show stronger modeling potential.

- `SGFormer` (Wu et al., 2023): a lightweight graph transformer that efficiently models large-scale graphs through global attention.
- `PolyNormer` (Deng et al., 2024): a graph transformer with polynomial expressiveness and efficient long-range modeling capability.

#### 3) Circuit-Specific Models

These models are designed for task-specific pain points in the EDA domain, with targeted strategies for issues such as circuit-data scarcity and label imbalance.

- `ParaGraph` (Ren et al., 2020): an early representative work in this area that predicts parasitic parameters using hierarchical graph-symbol techniques.
- `CirGPS` (Shen et al., 2025c): a circuit-specific model that addresses circuit-data scarcity through subgraph sampling and few-shot/pretraining strategies.
- `CircuitGCL` (Shen et al., 2025a): a circuit-specific graph-contrastive framework designed to improve representation quality under challenging circuit-data distributions.

### 3.3 Comparative Performance Results

Performance results of different models on SRAM circuits:

### Performance of Different Models on SRAM Circuits Ground Capacitance Node Regression Task

| Metric | sram+digtime+timing_ctrl MAE ↓ | sram+digtime+timing_ctrl R² ↑ | sandwich MAE ↓ | sandwich R² ↑ | ultra8t MAE ↓ | ultra8t R² ↑ | array_128_32_8t MAE ↓ | array_128_32_8t R² ↑ |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| GCN | 0.0461 | 0.8401 | 0.1876 | 0.1636 | 0.0192 | 0.5446 | 0.1199 | -0.1967 |
| GAT | 0.0454 | 0.8469 | 0.1838 | 0.2159 | 0.0955 | 0.5060 | 0.1211 | -0.2518 |
| GraphSAGE | 0.0413 | 0.8800 | 0.1964 | 0.0585 | 0.1093 | 0.2620 | 0.2813 | -2.0157 |
| PNA | 0.0415 | 0.8813 | 0.1845 | 0.3589 | 0.0894 | 0.4561 | 0.2024 | -0.9577 |
| SGFormer | 0.0424 | 0.8729 | 0.2297 | -0.1733 | 0.1619 | -0.3985 | 0.2934 | -3.2101 |
| PolyNormer | 0.0423 | 0.8667 | 0.1938 | 0.1186 | 0.1563 | -0.5618 | 0.1699 | -1.1344 |
| Paragraph | 0.1618 | 0.1487 | 0.1659 | 0.1795 | 0.1571 | 0.1863 | 0.1459 | 0.1973 |
| CirGPS | 0.0063 | 0.9568 | 0.0298 | 0.6574 | 0.0222 | 0.7882 | 0.0194 | 0.8702 |
| CircuitGCL | 0.0511 | 0.8792 | 0.3558 | -0.4768 | 0.3359 | -0.3439 | 0.3314 | -0.6812 |

Performance results of different models on the analog ground-capacitance node regression task:

### Performance of Different Models on Analog Circuits Ground Capacitance Node Regression Task

| Metric | 1-4, 6, 8-12, 15-18 MAE ↓ | 1-4, 6, 8-12, 15-18 R² ↑ | 5 MAE ↓ | 5 R² ↑ | 14 MAE ↓ | 14 R² ↑ | 20 MAE ↓ | 20 R² ↑ |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| GCN | 0.0272 | 0.9570 | 0.0171 | 0.4200 | 0.0216 | 0.8460 | 0.0284 | 0.8632 |
| GAT | 0.0367 | 0.9336 | 0.0078 | 0.4879 | 0.0107 | 0.8749 | 0.0230 | 0.8270 |
| GraphSAGE | 0.0473 | 0.8861 | 0.0267 | 0.3098 | 0.0311 | 0.8106 | 0.0371 | 0.7869 |
| PNA | 0.0292 | 0.9502 | 0.0089 | 0.4214 | 0.0094 | 0.8507 | 0.0211 | 0.8423 |
| SGFormer | 0.0798 | 0.7283 | 0.0340 | -0.0393 | 0.0339 | 0.5936 | 0.0571 | 0.4162 |
| PolyNormer | 0.0298 | 0.9667 | 0.0155 | 0.3828 | 0.0148 | 0.8638 | 0.0281 | 0.7959 |
| Paragraph | 0.1799 | -0.0527 | 0.0250 | -0.0862 | 0.0391 | -0.0008 | 0.0594 | -0.0170 |
| CirGPS | 0.0783 | 0.8384 | 0.0932 | -2.4965 | 0.1034 | -0.3204 | 0.1100 | 0.1395 |
| CircuitGCL | 0.0386 | 0.9605 | 0.0225 | 0.6119 | 0.0227 | 0.8679 | 0.0299 | 0.8126 |

## 4. Additional Regression Task Results

Performance of different models on the SRAM coupling-capacitance edge regression task:

### Performance of Different Models on SRAM Circuits Coupling Capacitance Edge Regression Task

| Metric | sram+digtime+timing_ctrl MAE ↓ | sram+digtime+timing_ctrl R² ↑ | sandwich MAE ↓ | sandwich R² ↑ | ultra8t MAE ↓ | ultra8t R² ↑ | array_128_32_8t MAE ↓ | array_128_32_8t R² ↑ |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| GCN | 0.0717 | 0.5008 | 0.1073 | 0.3511 | 0.0971 | 0.2866 | 0.0960 | 0.2543 |
| GAT | 0.0717 | 0.4960 | 0.1075 | 0.3504 | 0.0962 | 0.2969 | 0.0956 | 0.2581 |
| GraphSAGE | 0.0747 | 0.4701 | 0.1078 | 0.3427 | 0.0920 | 0.3334 | 0.0783 | 0.4182 |
| PNA | 0.0715 | 0.5053 | 0.1067 | 0.3527 | 0.0935 | 0.3083 | 0.0863 | 0.3554 |
| SGFormer | 0.0766 | 0.4592 | 0.1081 | 0.3212 | 0.0930 | 0.3093 | 0.0746 | 0.4378 |
| PolyNormer | 0.0709 | 0.5053 | 0.1053 | 0.3449 | 0.0934 | 0.2951 | 0.0835 | 0.3797 |
| Paragraph | 0.0884 | 0.3586 | 0.1127 | 0.3389 | 0.0961 | 0.3288 | 0.0783 | 0.3981 |
| CirGPS | 0.0714 | 0.4413 | 0.1093 | 0.3412 | 0.0933 | 0.3284 | 0.0723 | 0.3979 |
| CircuitGCL | 0.0600 | 0.6234 | 0.1109 | 0.3229 | 0.1005 | 0.2507 | 0.0886 | 0.2650 |

Performance of different models on the analog effective-resistance edge regression task:

### Performance of Different Models on Analog Circuits Effective Resistance Edge Regression Task

| Metric | 1-4, 6, 8-12, 15-18 MAE ↓ | 1-4, 6, 8-12, 15-18 R² ↑ | 5 MAE ↓ | 5 R² ↑ | 14 MAE ↓ | 14 R² ↑ | 20 MAE ↓ | 20 R² ↑ |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| GCN | 0.0587 | 0.8018 | 0.0637 | 0.4772 | 0.1513 | -0.5502 | 0.1687 | 0.6526 |
| GAT | 0.0550 | 0.8159 | 0.0717 | 0.4891 | 0.1533 | -0.6171 | 0.1871 | 0.5711 |
| GraphSAGE | 0.0461 | 0.8789 | 0.0511 | 0.7451 | 0.1473 | -0.5093 | 0.1396 | 0.7057 |
| PNA | 0.0439 | 0.9051 | 0.0408 | 0.8565 | 0.1031 | 0.2984 | 0.1463 | 0.7209 |
| SGFormer | 0.0489 | 0.8721 | 0.0397 | 0.8127 | 0.1224 | 0.0443 | 0.1439 | 0.6474 |
| PolyNormer | 0.0407 | 0.9035 | 0.0575 | 0.6137 | 0.0976 | 0.3099 | 0.1314 | 0.6991 |
| Paragraph | 0.0077 | 0.7999 | 0.0536 | 0.7306 | 0.1104 | 0.1104 | 0.2853 | -0.0742 |
| CirGPS | 0.0769 | 0.7443 | 0.0719 | 0.6890 | 0.2027 | 0.5314 | 0.0992 | 0.3049 |
| CircuitGCL | 0.0885 | 0.6410 | 0.2365 | -1.3683 | 0.1414 | -0.4614 | 0.2088 | 0.4782 |

Performance of different models on the SRAM effective-resistance node classification task:

### Performance of Different Models on SRAM Circuits Effective Resistance Node Classification Task

| Metric | sram+digtime+timing_ctrl Accuracy ↑ | sram+digtime+timing_ctrl F1-Score ↑ | sandwich Accuracy ↑ | sandwich F1-Score ↑ | ultra8t Accuracy ↑ | ultra8t F1-Score ↑ | array_128_32_8t Accuracy ↑ | array_128_32_8t F1-Score ↑ |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| GCN | 0.9608 | 0.9339 | 0.5738 | 0.5146 | 0.2440 | 0.1961 | 0.7789 | 0.4610 |
| GAT | 0.9528 | 0.9217 | 0.5273 | 0.3453 | 0.9534 | 0.4979 | 0.9951 | 0.8528 |
| GraphSAGE | 0.9529 | 0.9224 | 0.5281 | 0.3462 | 0.9863 | 0.4966 | 0.9953 | 0.8555 |
| PNA | 0.9771 | 0.9563 | 0.6011 | 0.5234 | 0.9145 | 0.4984 | 0.9978 | 0.8132 |
| SGFormer | 0.8003 | 0.7540 | 0.5280 | 0.3456 | 0.9982 | 0.4996 | 0.9951 | 0.8489 |
| PolyNormer | 0.8177 | 0.7726 | 0.5105 | 0.3807 | 0.9416 | 0.4849 | 0.9951 | 0.8497 |
| Paragraph | 0.8132 | 0.7412 | 0.4421 | 0.3142 | 0.9231 | 0.4357 | 0.9856 | 0.8259 |
| CirGPS | 0.7832 | 0.7211 | 0.5692 | 0.4011 | 0.8250 | 0.4732 | 0.9989 | 0.8336 |
| CircuitGCL | 0.8842 | 0.8734 | 0.9918 | 0.4980 | 0.4385 | 0.3691 | 0.9987 | 0.4997 |

## 5. Analog Topology-to-Graph

![Analog2Graph](IMGS/analog2graph.png)

The conversion from analog circuit schematics to graph representations follows the framework shown in the figure. We model each circuit as a heterogeneous graph $\mathcal{G}=(\mathcal{V},\mathcal{E})$. The node set $\mathcal{V}$ contains three types of nodes: *device nodes* representing circuit components, *net nodes* representing interconnect wires, and *pin nodes* representing device terminals. The topological edges $\mathcal{E}_{\text{topo}}$ (shown as black lines) capture circuit connectivity derived from the schematic, specifically through *device-to-pin* and *pin-to-net* connections; these topological relations constitute the input structure obtained from the schematic-to-graph transformation.

In contrast, parasitic information is obtained from the extracted parasitic netlist. Blue *pin-to-pin* edges are treated as resistive edges, where the label corresponds to the effective resistance between two pins (see the algorithm in Section 6). In addition, we assign the total ground capacitance of each net as a node-level label on the corresponding net node. These parasitic labels serve as prediction targets in our benchmark.

## 6. Algorithms

### 6.1 Matrix-Based Effective Resistance Calculation

> **Purpose.** Compute port-to-port effective resistance efficiently from the resistor netlist by constructing the nodal admittance matrix and querying pairwise resistances through its Cholesky factorization.

#### Overview

The procedure consists of three stages:

1. **Build the nodal admittance matrix**
   - Traverse all resistor elements in the net.
   - Convert each resistance value `r` into conductance `g = 1 / r`.
   - Update diagonal and off-diagonal entries according to Kirchhoff's Current Law (KCL).

2. **Construct the reduced invertible matrix**
   - Select one reference node, usually the ground node.
   - Remove the corresponding row and column from the admittance matrix.
   - Obtain the reduced admittance matrix `G_red`.

3. **Query effective resistances between ports**
   - Compute the Cholesky factor `L` such that `G_red = L L^T`.
   - Use `Z = L^{-1}` to derive pairwise effective resistances.
   - For each port pair, compute `R_eq = ||z_src - z_dst||^2`.

#### Pseudocode

<details>
<summary><strong>Algorithm 1. Matrix-Based Effective Resistance Calculation</strong></summary>

<br>

**Input:** Resistor list `R` of a net, port list `P`  
**Output:** Effective resistance list `L_out = {(src, dst, val)}`

```text
1:  L_out <- ∅
2:  V <- ExtractUniqueNodes(R)
3:  N <- |V|
4:  if N < 2 or |P| < 2 then
5:      return ∅
6:  end if
7:  M <- MapNodesToIndices(V)

8:  // Stage 1: Build admittance matrix
9:  G <- 0_{N×N}
10: for each (n1, n2, r) in R do
11:     g <- 1 / r
12:     u <- M[n1], v <- M[n2]
13:     G[u,u] <- G[u,u] + g
14:     G[v,v] <- G[v,v] + g
15:     G[u,v] <- G[u,v] - g
16:     G[v,u] <- G[v,u] - g
17: end for

18: ref <- N - 1
19: G_red <- G[0:ref, 0:ref]

20: // Stage 2: Cholesky factorization
21: Compute L such that G_red = L L^T
22: Z <- L^{-1}

23: // Stage 3: Port-to-port resistance extraction
24: for k <- 0 to |P| - 1 do
25:     for l <- k + 1 to |P| - 1 do
26:         src_id <- P[k]
27:         dst_id <- P[l]
28:         z_src <- column M[src_id] of Z
29:         z_dst <- column M[dst_id] of Z
30:         R_eq <- ||z_src - z_dst||^2
31:         append (src_id, dst_id, R_eq) to L_out
32:     end for
33: end for
34: return L_out
```

</details>

## 7. Limitations

Although ParasGB fills an important gap in benchmark research for circuit parasitic-effect modeling, several aspects still leave room for improvement as part of this early-stage exploration.

**Insufficient circuit-type coverage.** The current ParasGB dataset mainly covers SRAM and specific analog circuit modules. While these are representative, they do not cover all industrial design scenarios. For example, the layout styles and interconnect logic of complex digital circuits and very-large-scale SoC systems differ significantly from those of analog circuits. As a result, models trained on the current dataset may experience substantial performance degradation when directly transferred to digital-circuit scenarios.

**Challenges in accurate regression prediction.** Parasitic parameters exhibit pronounced long-tailed distributions, which makes extreme-value samples (very large or very small values) difficult to predict accurately. Although discretization (binning) reduces the difficulty of the task, it is essentially a compromise. Industrial applications still require high-precision numerical regression, and meeting that demand remains a core challenge for current algorithms.

**Lack of cross-technology-node validation.** The current dataset is mainly derived from specific advanced technologies. However, physical properties and design rules vary significantly across semiconductor technology generations, such as from 28 nm to 5 nm. Without large-scale cross-technology comparison data, it is difficult to fully validate model transferability to new technology nodes, which limits generalization across different foundries.

**Limited depth of physical-interaction modeling.** Current node features mainly include spatial coordinates and device-size information. In real chips, however, deeper physical effects such as local thermal behavior and complex electromagnetic coupling across multiple metal layers can also affect parasitic parameters. Although existing graph structures can model topological connectivity, they still provide insufficient depth for modeling these three-dimensional physical interactions, and some key physical features may therefore be overlooked.

## 8. Future Directions

### 1. Graph Foundation Model Pretraining

Conduct self-supervised pretraining on large-scale circuit graphs to learn general physical laws of circuits, and then adapt to downstream tasks with limited fine-tuning.

### 2. Enhanced Spatial-Geometric Awareness

Incorporate 3D layout information and metal-layer attributes more deeply into message passing so that the model can jointly understand:

- topological connectivity
- three-dimensional relative position
- routing-coupling relationships

### 3. Building a More Comprehensive Evaluation Platform

Future versions are planned to include:

- RF circuits
- high-speed interfaces
- large-scale digital logic modules
- layout data across multiple process nodes

### 4. Real-Time Guidance for Design Closure

Move from an offline benchmark toward online design assistance by providing parasitic warnings and optimization suggestions during the layout stage.
