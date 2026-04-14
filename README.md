# ParasGB: Appendix Supplementary Materials

## 📑 Table of Contents

- [Appendix A: User Guide](#appendix-a-user-guide)
- [Appendix B: Dataset Details](#appendix-b-dataset-details)
- [Appendix C: Experimental Settings](#appendix-c-experimental-settings)
- [Appendix D: Regression Results](#appendix-d-regression-results)
- [Appendix G: Analog-Specific Graph Construction](#appendix-g-analog-specific-graph-construction)
- [Appendix H: Effective Resistance Calculation](#appendix-h-effective-resistance-calculation)

## Appendix A: User Guide

A complete Python example from data download to calling the standard evaluator is provided in the `docs/` directory.

### Data Loading Example

```python
from parasgb.data import load_dataset

# Load analog circuit dataset
analog_data = load_dataset('analog', scale='S')

# Load SRAM circuit dataset
sram_data = load_dataset('sram', scale='L')

# Access graph attributes
print(f"Number of nodes: {analog_data.num_nodes}")
print(f"Number of edges: {analog_data.num_edges}")
print(f"Node features shape: {analog_data.x.shape}")
print(f"Edge features shape: {analog_data.edge_attr.shape}")
```

### Model Training Example

```python
from parasgb.models import GCN
from parasgb.training import train_model

# Initialize model
model = GCN(
    in_channels=analog_data.x.shape[1],
    hidden_channels=256,
    out_channels=1,  # Regression task
    num_layers=4,
    dropout=0.3
)

# Train model
trained_model = train_model(
    model=model,
    data=analog_data,
    task='node_cg',
    epochs=100,
    batch_size=128,
    learning_rate=0.001
)

# Save model
trained_model.save('models/best_model.pth')
```

### Evaluation Example

```python
from parasgb.evaluation import evaluate_model

# Evaluate model on test set
results = evaluate_model(
    model_path='models/best_model.pth',
    test_data='data/processed/test_data.pt',
    task='node_cg'
)

print(f"Accuracy: {results['accuracy']:.4f}")
print(f"F1-Score: {results['f1_score']:.4f}")
print(f"MAE: {results['mae']:.6e}")
print(f"R² Score: {results['r2']:.4f}")

# Generate predictions
predictions = evaluate_model(
    model_path='models/best_model.pth',
    test_data='data/processed/test_data.pt',
    task='node_cg',
    return_predictions=True
)

print(f"Predictions shape: {predictions.shape}")
```

## Appendix B: Dataset Details

### Label Distributions

Detailed histograms of label distributions for all 20 analog circuits and 6 SRAM subsets are provided in the `figures/` directory:

#### Analog Circuits
- **Ground Capacitance (Cg)**: Most values range from 10⁻¹⁴ F to 10⁻¹³ F, with a mean of ~8.9 × 10⁻¹⁴ F
- **Effective Resistance (Reff)**: Values span from 0.01 Ω to over 600 Ω, with a mean of ~97.4 Ω
- **Edge Count**: Average of ~2978 edges per circuit

**Figure References:**
- `figures/analog_cg_distribution.png` - Ground capacitance distribution across analog circuits
- `figures/analog_reff_distribution.png` - Effective resistance distribution across analog circuits
- `figures/analog_edge_distribution.png` - Edge count distribution across analog circuits

#### SRAM Circuits
- **Ground Capacitance (Cg)**: Ranges from 10⁻¹⁹ F to 10⁻¹² F, with a mean of ~1.26 × 10⁻¹⁶ F
- **Coupling Capacitance (Cc)**: Ranges from 10⁻²⁷ F to 10⁻¹¹ F, with a mean of ~1.12 × 10⁻¹⁷ F
- **Effective Resistance (Reff)**: Ranges from 2.58 Ω to 3009.74 Ω, with a mean of ~252.21 Ω

**Figure References:**
- `figures/sram_cg_distribution.png` - Ground capacitance distribution across SRAM circuits
- `figures/sram_cc_distribution.png` - Coupling capacitance distribution across SRAM circuits
- `figures/sram_reff_distribution.png` - Effective resistance distribution across SRAM circuits

### Node and Edge Statistics

| Circuit Type | Average Nodes | Average Edges | Device Nodes | Pin Nodes | Net Nodes |
|--------------|---------------|---------------|--------------|-----------|-----------|
| Analog (XS)  | 342           | 834           | 68           | 156       | 118       |
| Analog (S)   | 1288          | 2637          | 382          | 800       | 106       |
| Analog (M)   | 4474          | 5535          | 1020         | 2235      | 1219      |
| SRAM (L)     | 17.9K         | 25.5K         | 4.2K         | 12.7K     | 1.0K      |
| SRAM (XL)    | 196K          | 270.5K        | 45.1K        | 133.5K    | 17.2K     |
| SRAM (XXL)   | 10.9M         | 14.8M         | 2.4M         | 7.4M      | 1.0M      |

**Figure References:**
- `figures/node_edge_statistics.png` - Node and edge count statistics across circuit types
- `figures/circuit_scale_comparison.png` - Comparison of circuit scales

### Feature Distribution Visualizations

Detailed feature distribution visualizations are provided in the `figures/` directory:

- **Device Features**: Type indicators and geometric parameters show distinct distributions across circuit types
  - `figures/device_feature_distribution.png` - Device feature distributions

- **Net Features**: Connectivity statistics reveal different wiring patterns between analog and SRAM circuits
  - `figures/net_feature_distribution.png` - Net feature distributions

- **Pin Features**: Terminal type distributions reflect the different device compositions in various circuit types
  - `figures/pin_feature_distribution.png` - Pin feature distributions

- **Feature Correlation**: Heatmaps showing correlations between different features
  - `figures/feature_correlation_matrix.png` - Feature correlation matrix

## Appendix C: Experimental Settings

### Hyperparameter Settings

| Model | Hidden Dimension | Number of Layers | Dropout | Learning Rate | Batch Size | Weight Decay |
|-------|-----------------|------------------|---------|---------------|------------|-------------|
| GCN   | 256             | 4                | 0.3     | 0.001         | 128        | 1e-5        |
| GAT   | 256             | 4                | 0.3     | 0.001         | 128        | 1e-5        |
| GraphSAGE | 256          | 4                | 0.3     | 0.001         | 128        | 1e-5        |
| PNA   | 256             | 4                | 0.3     | 0.001         | 128        | 1e-5        |
| SGFormer | 256          | 4                | 0.3     | 0.001         | 128        | 1e-5        |
| PolyNormer | 256         | 4                | 0.3     | 0.001         | 128        | 1e-5        |
| ParaGraph | 256          | 4                | 0.3     | 0.001         | 128        | 1e-5        |
| CircuitGPS | 256          | 4                | 0.3     | 0.001         | 128        | 1e-5        |
| CircuitGCL | 256          | 4                | 0.3     | 0.001         | 128        | 1e-5        |

**Figure References:**
- `figures/hyperparameter_tuning.png` - Hyperparameter tuning results

### Training Strategies

- **Optimizer**: AdamW
- **Learning Rate Scheduler**: Cosine annealing with warm restarts (T_0=10, T_mult=2)
- **Early Stopping**: Patience of 20 epochs
- **Loss Functions**:
  - Classification: Cross-entropy loss with class weights inversely proportional to class frequencies
  - Regression: Mean absolute error (MAE) with Huber loss for robustness to outliers
- **Validation Strategy**: 10% of training data used for validation
- **Data Augmentation**:
  - Node feature perturbation (±5%)
  - Edge dropout (5%)
  - Random node masking (3%)

**Figure References:**
- `figures/training_curves.png` - Training and validation loss curves
- `figures/learning_rate_schedule.png` - Learning rate schedule visualization

### Hardware Configuration

- **GPUs**: NVIDIA A100 80GB (for large SRAM circuits), NVIDIA V100 32GB (for analog circuits)
- **CPUs**: Intel Xeon 8375C (32 cores, 2.9 GHz)
- **Memory**: 256GB RAM
- **Storage**: NVMe SSD (1TB) for fast data access
- **Software Environment**:
  - Python 3.9.10
  - PyTorch 2.0.0
  - PyTorch Geometric 2.3.0
  - CUDA 11.7
  - cuDNN 8.5.0

**Figure References:**
- `figures/hardware_utilization.png` - Hardware utilization during training

## Appendix D: Regression Results

### Analog Circuits

#### Ground Capacitance (Cg) Regression

| Model | MAE (F) | MSE (F²) | R² Score |
|-------|---------|----------|----------|
| GCN   | 1.23e-14 | 2.15e-28 | 0.78     |
| GAT   | 1.18e-14 | 1.98e-28 | 0.80     |
| GraphSAGE | 1.15e-14 | 1.87e-28 | 0.81 |
| PNA   | 1.02e-14 | 1.45e-28 | 0.85     |
| SGFormer | 1.10e-14 | 1.72e-28 | 0.83 |
| PolyNormer | 1.05e-14 | 1.54e-28 | 0.84 |
| ParaGraph | 1.35e-14 | 2.48e-28 | 0.75 |
| CircuitGPS | 1.28e-14 | 2.27e-28 | 0.77 |
| CircuitGCL | 1.12e-14 | 1.78e-28 | 0.82 |

**Figure References:**
- `figures/analog_cg_regression_results.png` - Ground capacitance regression results for analog circuits
- `figures/analog_cg_prediction_scatter.png` - Predicted vs. actual ground capacitance for analog circuits

#### Effective Resistance (Reff) Regression

| Model | MAE (Ω) | MSE (Ω²) | R² Score |
|-------|---------|----------|----------|
| GCN   | 45.2    | 3256     | 0.62     |
| GAT   | 47.8    | 3582     | 0.59     |
| GraphSAGE | 46.5    | 3421     | 0.60 |
| PNA   | 49.3    | 3745     | 0.57     |
| SGFormer | 44.1    | 3128     | 0.63 |
| PolyNormer | 51.2    | 3987     | 0.55 |
| ParaGraph | 48.7    | 3654     | 0.58 |
| CircuitGPS | 43.5    | 3052     | 0.64 |
| CircuitGCL | 46.8    | 3456     | 0.60 |

**Figure References:**
- `figures/analog_reff_regression_results.png` - Effective resistance regression results for analog circuits
- `figures/analog_reff_prediction_scatter.png` - Predicted vs. actual effective resistance for analog circuits

### SRAM Circuits

#### Ground Capacitance (Cg) Regression

| Model | MAE (F) | MSE (F²) | R² Score |
|-------|---------|----------|----------|
| GCN   | 3.2e-17 | 1.6e-33 | 0.65     |
| GAT   | 3.4e-17 | 1.8e-33 | 0.63     |
| GraphSAGE | 3.1e-17 | 1.5e-33 | 0.66 |
| PNA   | 2.8e-17 | 1.2e-33 | 0.70     |
| SGFormer | 3.0e-17 | 1.4e-33 | 0.68 |
| PolyNormer | 2.9e-17 | 1.3e-33 | 0.69 |
| ParaGraph | 3.8e-17 | 2.2e-33 | 0.59 |
| CircuitGPS | 1.5e-17 | 0.4e-33 | 0.85     |
| CircuitGCL | 2.7e-17 | 1.1e-33 | 0.71 |

**Figure References:**
- `figures/sram_cg_regression_results.png` - Ground capacitance regression results for SRAM circuits
- `figures/sram_cg_prediction_scatter.png` - Predicted vs. actual ground capacitance for SRAM circuits

#### Coupling Capacitance (Cc) Regression

| Model | MAE (F) | MSE (F²) | R² Score |
|-------|---------|----------|----------|
| GCN   | 1.2e-18 | 2.1e-36 | 0.72     |
| GAT   | 1.1e-18 | 1.9e-36 | 0.73     |
| GraphSAGE | 1.3e-18 | 2.3e-36 | 0.71 |
| PNA   | 1.0e-18 | 1.5e-36 | 0.75     |
| SGFormer | 1.1e-18 | 1.8e-36 | 0.74 |
| PolyNormer | 1.0e-18 | 1.4e-36 | 0.76     |
| ParaGraph | 0.8e-18 | 1.0e-36 | 0.82     |
| CircuitGPS | 1.4e-18 | 2.5e-36 | 0.70 |
| CircuitGCL | 1.5e-18 | 2.7e-36 | 0.69 |

**Figure References:**
- `figures/sram_cc_regression_results.png` - Coupling capacitance regression results for SRAM circuits
- `figures/sram_cc_prediction_scatter.png` - Predicted vs. actual coupling capacitance for SRAM circuits

## Appendix G: Analog-Specific Graph Construction

### Device Modeling

- **MOSFETs**: Represented with width (W), length (L), multiplier (M), finger count (Nf), and device type (T)
  - **Width (W)**: Channel width in microns
  - **Length (L)**: Channel length in microns
  - **Multiplier (M)**: Number of parallel devices
  - **Finger Count (Nf)**: Number of fingers for multi-finger devices
  - **Type (T)**: Device type (e.g., nmos, pmos)

- **Resistors**: Modeled with resistance value, geometry, and type
  - **Resistance Value**: Resistance in ohms
  - **Geometry**: Length and width in microns
  - **Type**: Resistor type (e.g., poly, diff)

- **Capacitors**: Represented with capacitance value, geometry, and type
  - **Capacitance Value**: Capacitance in farads
  - **Geometry**: Area and perimeter in microns
  - **Type**: Capacitor type (e.g., moscap, polycap)

**Figure References:**
- `figures/device_modeling.png` - Device modeling diagram
- `figures/mosfet_parameters.png` - MOSFET parameter visualization

### Parasitic Extraction

- **Local Extraction**: Captures device-level parasitics
  - **Source/Drain Capacitance**: Parasitic capacitance at device terminals
  - **Gate Capacitance**: Overlap and fringing capacitance at the gate
  - **Body Capacitance**: Capacitance between device body and other terminals

- **Global Extraction**: Models interconnect parasitics
  - **Metal Line Resistance**: Resistance of interconnect wires
  - **Metal Line Capacitance**: Capacitance between metal lines and substrate
  - **Via Resistance/Capacitance**: Parasitics associated with vias

- **RC Network Simplification**: Reduces complexity while preserving critical path delays
  - **Lumping**: Groups distributed parasitics into lumped elements
  - **Tree Pruning**: Removes non-critical branches from the RC network
  - **Equivalent Circuit Generation**: Creates simplified equivalent circuits

**Figure References:**
- `figures/parasitic_extraction_flow.png` - Parasitic extraction flowchart
- `figures/rc_network_simplification.png` - RC network simplification process

### Edge Construction

- **Device-to-Pin Edges**: Represent physical connections between devices and their terminals
  - **Direction**: From device to pin
  - **Attributes**: Terminal type (gate, source, drain, body)

- **Pin-to-Net Edges**: Model connectivity between terminals and nets
  - **Direction**: From pin to net
  - **Attributes**: Connection type (signal, power, ground)

- **Parasitic Edges**: Generated as prediction targets for capacitance and resistance
  - **Capacitance Edges**: Represent coupling capacitance between nets
  - **Resistance Edges**: Represent effective resistance between pins

**Figure References:**
- `figures/graph_construction.png` - Graph construction process
- `figures/edge_types.png` - Different edge types in the circuit graph

## Appendix H: Effective Resistance Calculation

### Methodology

1. **Resistive Network Identification**: Extracts resistive sub-networks from the full circuit
   - **Node Identification**: Identifies nodes connected by resistive elements
   - **Network Extraction**: Extracts resistive sub-networks from the full circuit
   - **Network Validation**: Ensures extracted networks are electrically meaningful

2. **Pin Pair Selection**: Randomly selects pin pairs within each resistive network
   - **Uniform Sampling**: Ensures representative coverage of the network
   - **Distance Consideration**: Balances short and long distance pairs
   - **Redundancy Avoidance**: Avoids redundant measurements

3. **Effective Resistance Computation**: Uses modified nodal analysis to calculate resistance between pin pairs
   - **Conductance Matrix Construction**: Builds the conductance matrix for the network
   - **Current Injection**: Applies a unit current between the pin pair
   - **Voltage Solution**: Solves for node voltages using linear algebra
   - **Resistance Calculation**: Computes resistance from voltage difference

4. **Validation**: Compares with SPICE simulation results to ensure accuracy
   - **SPICE Simulation**: Runs SPICE simulations for the same pin pairs
   - **Error Calculation**: Computes error between analytical and simulation results
   - **Threshold Checking**: Ensures errors are within acceptable bounds

**Figure References:**
- `figures/effective_resistance_calculation.png` - Effective resistance calculation methodology
- `figures/resistive_network_extraction.png` - Resistive network extraction process

### Algorithm

```python
def compute_effective_resistance(network, pin1, pin2):
    """
    Compute effective resistance between two pins in a resistive network
    
    Args:
        network: Resistive network object
        pin1: Index of first pin
        pin2: Index of second pin
        
    Returns:
        Reff: Effective resistance between pin1 and pin2 in ohms
    """
    # 1. Build conductance matrix
    num_nodes = len(network.nodes)
    G = np.zeros((num_nodes, num_nodes))
    
    for edge in network.edges:
        i, j = edge.nodes
        R = edge.resistance
        G[i, j] -= 1/R
        G[j, i] -= 1/R
        G[i, i] += 1/R
        G[j, j] += 1/R
    
    # 2. Apply current source between pin1 and pin2
    I = np.zeros(num_nodes)
    I[pin1] = 1.0  # Inject 1A at pin1
    I[pin2] = -1.0 # Extract 1A at pin2
    
    # 3. Solve for node voltages
    # Remove one row and column to account for ground reference
    G_reduced = np.delete(np.delete(G, pin2, axis=0), pin2, axis=1)
    I_reduced = np.delete(I, pin2)
    V_reduced = np.linalg.solve(G_reduced, I_reduced)
    
    # Reconstruct full voltage vector
    V = np.zeros(num_nodes)
    V[:pin2] = V_reduced[:pin2]
    V[pin2+1:] = V_reduced[pin2:]
    V[pin2] = 0.0  # Ground reference
    
    # 4. Calculate effective resistance
    Reff = V[pin1] - V[pin2]
    
    return Reff
```

### Validation Results

| Circuit Type | Average Error | Maximum Error | Standard Deviation |
|--------------|---------------|---------------|-------------------|
| Analog       | 2.3%          | 7.8%          | 1.5%              |
| SRAM         | 1.8%          | 5.2%          | 1.1%              |
| Combined     | 2.1%          | 7.8%          | 1.3%              |

**Figure References:**
- `figures/effective_resistance_validation.png` - Effective resistance validation results
- `figures/error_distribution.png` - Error distribution across circuit types

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.
