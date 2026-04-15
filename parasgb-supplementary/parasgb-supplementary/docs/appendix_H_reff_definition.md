# Appendix H — Effective Resistance (`Reff`) Definition and Computation

## Purpose of this document

This appendix explains how ParasGB defines and computes edge-level effective resistance labels. Since `Reff` is not simply a raw resistor value copied from the extracted netlist, this page is important for readers who want to understand exactly what the supervision target means.

---

## H.1 Why `Reff` is needed

In extracted post-layout circuits, a logical connection is often represented by a distributed resistive subnetwork rather than by a single resistor. If the benchmark were to use raw extracted resistor elements directly, the labels would be hard to interpret and much less suitable for scalable graph learning.

ParasGB therefore uses **effective resistance** between selected node pairs (or pin pairs) to summarize the resistive behavior of a subnetwork in a physically meaningful way.

---

## H.2 Intuitive interpretation

The effective resistance between two terminals measures the equivalent resistance observed between them when the full resistive network is taken into account.

This is more informative than reading off one local resistor, because it reflects:

- all available conductive paths,
- the topology of the resistive network,
- the aggregate electrical relationship between the two endpoints.

In the benchmark, `Reff` is used as an edge-level prediction target.

---

## H.3 Input objects for the algorithm

A practical description of the released procedure can be written in terms of:

- a resistor list `R` for a given net or resistive component graph,
- a port list `P` containing the terminal or pin identifiers of interest.

From these inputs, the algorithm outputs a list of tuples such as:

```text
(src, dst, value)
```

where `value` is the effective resistance between `src` and `dst`.

---

## H.4 Matrix-based formulation

The main idea is to construct a node-admittance matrix for the resistive network.

### Step 1: Build the admittance matrix

For each resistor `(n1, n2, r)`:

- convert resistance to conductance using `g = 1 / r`,
- update the diagonal entries of the admittance matrix,
- update the off-diagonal entries using the usual conductance-based network formulation.

### Step 2: Choose a reference node

To make the matrix invertible, remove one reference node (often corresponding to ground or another designated reference).

### Step 3: Use the Cholesky factorization

Factor the reduced admittance matrix and compute the inverse of the Cholesky factor.

### Step 4: Query pairwise effective resistance

For each selected pair of ports, compute the effective resistance from the transformed representation derived from the factorization.

This avoids repeated expensive direct simulation for every queried pair.

---

## H.5 Why this algorithm is suitable for the benchmark

This matrix-based formulation is useful for ParasGB because it is:

- more scalable than repeated path-based or brute-force circuit queries,
- compatible with large numbers of labels,
- physically grounded,
- suitable for generating many edge-level supervision targets offline.

This is especially important when the benchmark contains very large graphs and many candidate edge labels.

---

## H.6 Suggested pseudocode section

You can keep the block below as prose or later replace it with your exact released pseudocode.

```text
Input: resistor list R, port list P
Output: effective-resistance list Lout

1. Extract unique nodes and map them to matrix indices.
2. Build the nodal admittance matrix G.
3. Remove one reference node to obtain an invertible reduced matrix.
4. Compute a Cholesky factor of the reduced matrix.
5. Invert the factor (or equivalently derive the transformed representation).
6. For every queried port pair, compute the corresponding effective resistance.
7. Store the results as benchmark labels.
```

---

## H.7 Practical release notes

This appendix is also the right place to explain details such as:

- how port pairs are sampled or selected,
- whether all pairs are kept or only a subset,
- whether disconnected cases are filtered out,
- how numerical stability is handled,
- which units are stored in the raw files,
- whether labels are later normalized before training.

---

## H.8 Suggested figure and table additions

### Recommended figure

A small workflow figure showing:

1. extracted resistor network,
2. node-admittance matrix construction,
3. factorization step,
4. final pairwise `Reff` labels.

**Placeholder figure:**
- `../figures/appendix/fig_H1_reff_algorithm_overview.png`

### Recommended table

A concise notation table for symbols such as:

- `R`,
- `P`,
- `G`,
- `Gred`,
- `L`,
- `Z`,
- `Req`.

**Placeholder table:**
- `../tables/analog/table_H1_reff_notation.csv`

---

## H.9 Suggested wording for the final public release

> Effective resistance labels in ParasGB are derived from extracted resistive subnetworks rather than from single local resistor elements. For each selected pair of terminals, the benchmark computes a physically meaningful equivalent resistance using a matrix-based method built on the reduced node-admittance matrix of the network.

This wording usually reads clearly both for EDA readers and for graph-learning readers.

---

## H.10 Release checklist for this appendix

Before release, make sure this page clarifies:

- [ ] what the endpoints of `Reff` edges are,
- [ ] how port pairs are chosen,
- [ ] how the admittance matrix is built,
- [ ] what reference node convention is used,
- [ ] whether labels are normalized downstream,
- [ ] where the final algorithm figure is stored.

---

## H.11 Placeholder links

Replace the placeholders below once files are added:

- Algorithm overview figure: `../figures/appendix/fig_H1_reff_algorithm_overview.png`
- Notation table: `../tables/analog/table_H1_reff_notation.csv`
- Optional implementation note: `../metadata/reff_generation_notes.md`
