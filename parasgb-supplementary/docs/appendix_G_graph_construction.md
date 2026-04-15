# Appendix G — Analog-Specific Graph Construction

## Purpose of this document

This appendix explains how analog circuit data are converted from schematic / extracted representations into the heterogeneous graph format used by ParasGB. The main paper describes the shared topology-to-graph pipeline at a high level; this page is where the analog-specific choices should be documented in enough detail for a reader to understand the released graph objects.

---

## G.1 Overview

The graph-construction pipeline starts from design information and extracted parasitic data, then produces a graph with:

- **device nodes**,
- **pin nodes**,
- **net nodes**,
- **topological edges** derived from circuit connectivity,
- **parasitic labels** that are *not* part of the input topology but are instead used as prediction targets.

For clarity, the benchmark should emphasize the distinction between:

1. the **input graph structure**, which comes from topology and feature engineering, and
2. the **supervision targets**, which come from post-layout parasitic extraction and simplification.

---

## G.2 Node types

### Device nodes

Device nodes represent circuit elements such as MOSFETs, resistors, and capacitors.

Typical attributes may include:

- device category,
- transistor geometry,
- multiplicity / finger count,
- optional process or role tags.

### Pin nodes

Pin nodes represent terminals attached to devices.

Typical attributes may encode terminal roles such as:

- drain,
- gate,
- source,
- bulk,
- capacitor terminal,
- resistor terminal.

### Net nodes

Net nodes represent interconnect wires or electrical nets.

Typical attributes may include:

- functional role flags,
- counts of attached terminals and devices,
- aggregated width / length descriptors,
- signal / power / ground indicators.

---

## G.3 Topological edges

The benchmark graph contains topological edges that describe circuit connectivity.

### Device-to-pin edges

These connect a device node to the terminal nodes belonging to that device.

### Pin-to-net edges

These connect terminal nodes to the net they touch.

Together, these two edge families form the structural input graph used by the model.

### Important clarification

Parasitic `Cc` and `Reff` targets should not be confused with these topological edges. They are labels defined from post-layout extraction and should be documented separately.

---

## G.4 From extracted parasitic netlists to learnable graph labels

Raw post-layout netlists can be too detailed to use directly for scalable graph learning. A single logical net may expand into a large distributed RC structure. ParasGB therefore simplifies the extracted representation into a lumped supervision format.

### Node-level label

- net node -> `Cg`

### Edge-level labels

- selected pin pair -> `Reff`
- corresponding net pair -> `Cc`

This simplification is meant to preserve the information most relevant to timing, noise, and parasitic behavior while keeping the benchmark computationally manageable.

---

## G.5 Why a separate analog-specific appendix is useful

Although SRAM and analog circuits share the same broad workflow, the analog subset often deserves additional explanation because:

- module types are more heterogeneous,
- graph sizes are smaller but the target magnitudes can be extremely small,
- topology semantics are often more design-specific,
- device attributes can play a more interpretable role in feature design.

This appendix is therefore the right place to show one or two concrete analog examples.

---

## G.6 Suggested figure layout for this appendix

A strong supplementary figure for this appendix usually has four panels:

1. **schematic view**,
2. **post-layout / extracted netlist view**,
3. **simplified lumped RC view**,
4. **final graph view**.

If possible, use one analog design as the running example and keep the naming consistent across all four panels.

**Placeholder figure:**
- `../figures/appendix/fig_G1_analog_graph_conversion.png`

---

## G.7 Suggested step-by-step description

You can keep the following outline and replace the placeholders with your precise internal terminology later.

### Step 1: Start from the design representation

Obtain the schematic-level connectivity and the post-layout extracted parasitic information for the analog circuit.

### Step 2: Build structural nodes

Create device, pin, and net nodes.

### Step 3: Build structural edges

Create device-to-pin and pin-to-net edges.

### Step 4: Extract labels

From the extracted RC representation, derive:

- net-level ground capacitance,
- pairwise coupling capacitance,
- effective resistance between relevant pin pairs.

### Step 5: Attach features and targets

Store structural features on nodes and define parasitic targets as benchmark labels.

---

## G.8 Optional implementation note

If you want this appendix to be more practical for readers, add a short pseudocode block describing the conversion pipeline. For example:

```text
read schematic
read extracted netlist
build device/pin/net nodes
build dev2pin and pin2net edges
aggregate parasitic quantities
attach Cg / Cc / Reff labels
export hetero-graph object
```

---

## G.9 Release checklist for this appendix

Before release, check that this page links to:

- [ ] one analog conversion figure,
- [ ] one node/edge type summary table,
- [ ] one optional pseudocode block or manifest note,
- [ ] one note clarifying what is omitted for confidentiality.

---

## G.10 Placeholder links

Replace the placeholders below once files are added:

- Analog graph conversion figure: `../figures/appendix/fig_G1_analog_graph_conversion.png`
- Analog node/edge schema table: `../tables/analog/table_G1_analog_graph_schema.csv`
- Graph export manifest: `../metadata/graph_format_notes.md`
