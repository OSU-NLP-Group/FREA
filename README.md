# RxnVerif: A Chemical Reaction Feasibility Verification Dataset

**RxnVerif** is a binary classification benchmark for chemical reaction feasibility prediction.
Given a set of reactants and a target product represented as SMILES strings, the task is to predict whether the specified transformation is chemically feasible.

---

## Task Definition

Given a set of reactants **R** and a target product **P**, the goal is to predict:

```
f(R, P) → {0, 1}
```

where `1` indicates a feasible reaction and `0` indicates an infeasible one.

Feasibility is defined at the level of **mechanistic plausibility**: a reaction is considered feasible if a chemically valid mechanistic pathway exists by which the atoms of **R** can be reorganized to form **P**, regardless of the specific reagents, solvents, or catalysts that may be required. Reagent and condition information is intentionally excluded from the input, as this benchmark targets the mechanistic assessment stage of retrosynthesis planning, where such information is not yet available.

---

## Dataset Overview

| Split     | Method                  | Label | Count       |
| --------- | ----------------------- | ----- | ----------- |
| Positive  | —                       | 1     | 50,000      |
| Negative  | Random Addition (RA)    | 0     | 50,000      |
| Negative  | Random Replacement (RR) | 0     | 50,000      |
| Negative  | New Center (NC)         | 0     | 50,000      |
| Negative  | Forward Template (FT)   | 0     | 50,000      |
| **Total** |                         |       | **250,000** |

---

## Dataset Construction

### Positive Samples

Positive reactions are sourced from USPTO-full, a large-scale dataset of experimentally recorded chemical reactions extracted from the patent literature. Starting from the full dataset, we apply the following curation pipeline:

- **Filtering**: Reactions with missing reactant or product SMILES are removed. For reactions with recorded yield, those with a yield of zero are discarded, as these represent experimentally failed transformations.
- **Canonicalization**: All SMILES strings are canonicalized using RDKit. Reactions that cannot be parsed or sanitized are discarded.
- **Deduplication**: Exact duplicate reactions are removed based on canonical SMILES comparison.
- **Reagent removal**: Reagent information is excluded from the input; only reactant and product SMILES are retained, consistent with the mechanistic feasibility formulation.

### Negative Sample Generation

A fundamental challenge in constructing a reaction feasibility benchmark is the near-total absence of naturally occurring negative examples: the chemical literature overwhelmingly records successful reactions, leaving infeasible ones largely undocumented. We address this by designing four complementary negative generation methods that perturb confirmed positive reactions in chemically motivated ways to produce infeasible counterparts. Each method targets a qualitatively different aspect of the reaction.

For each positive reaction, we first extract the **reaction center** — the set of atoms involved in bond changes, identified via atom mapping — and decompose each reactant into its **synthon** by cleaving at the boundary between the reaction-center fragment and the surrounding non-reactive scaffold. These intermediate representations are used by several of the methods below.

#### Random Addition (RA)

RA constructs negatives by chemically modifying reactants at their reaction centers, producing species incompatible with the bond-forming event required to yield the original product. Each reactant is first decomposed into its synthon by removing leaving groups. New atoms, drawn from a frequency-weighted distribution of common leaving-group atoms (predominantly C, O, N, and halogens), are then appended one by one onto the reaction center or previously appended atoms, subject to valence constraints. The modified synthons are reassembled into full reactants by restoring the surrounding scaffold. The resulting combination of modified reactants and the original product is labeled as a negative sample.

#### Random Replacement (RR)

RR replaces one or more reactants with structurally similar but functionally distinct molecules drawn from the positive training corpus. For each positive reaction, the top-200 most similar reactions are retrieved by Tanimoto similarity computed on the joint reactant–product fingerprint. From these, a reaction is selected whose reactants are structurally close to the original but chemically incompatible with producing the original product. The combination of the replacement reactants and the original product is labeled as a negative sample.

#### New Center (NC)

NC generates negatives by identifying an alternative retrosynthetic disconnection of the product — one that differs from the true reaction center — and completing the resulting synthons into full reactants. Candidate reaction centers are predicted using [G2Retro](https://github.com/ninglab/G2Retro), retaining up to three predictions whose identified centers differ from the ground-truth center. The corresponding synthons are passed to [RLSynC](https://github.com/ninglab/RLSynC), an RL-based synthon completion model, with low-ranked completions deliberately selected to obtain molecules that are structurally plausible yet unlikely to correspond to known synthetic pathways. The completed reactants are paired with the original product to form a negative sample.

#### Forward Template (FT)

FT generates negatives by applying forward reaction templates to the original reactants, producing alternative products that could plausibly arise from the same starting materials but were not observed. For each positive reaction, all templates occurring at least 10 times in the training corpus — as well as a random sample of low-frequency templates — are applied to the original reactants. Any resulting product that differs from the ground-truth product under canonical SMILES comparison and does not appear as a known product of the same reactants in the positive data is paired with the original reactants to form a negative sample.

---

## Data Format

The dataset is provided as a CSV file (`data/rxnverif_v1.0.csv`) with the following columns:

| Column      | Description                                                  |
| ----------- | ------------------------------------------------------------ |
| `reactants` | Dot-separated SMILES of reactant molecules (canonicalized)   |
| `product`   | SMILES of the target product (canonicalized)                 |
| `method`    | Generation method: `no_method` for positives; `RA`, `RR`, `NC`, or `FT` for negatives |
| `feasible`  | Binary label: `1` = feasible, `0` = infeasible               |

**Example:**

```csv
reactants,product,method,feasible
C#CC(C)(CCC(C)=C(C)C)OC(C)=O,CCC(C)(CCC(C)=C(C)C)OC(C)=O,no_method,1
C#CC(OC)=C1OC1CCl.NOC1=CC=CC=C1,OC(CCl)COC1=CC=CC=C1,RA,0
```

---

## Citation

If you use this dataset in your research, please cite:

```bibtex
@misc{yu2025rxnverif,
  author       = {Yu, Botao and Adu-Ampratwum, Daniel and Zhou, Bo and Baker, Frazier N. and Chen, Ziru and Gao, Wenhao and Liu, Ye and Ning, Xia and Sun, Huan},
  title        = {{RxnVerif}: A Chemical Reaction Feasibility Verification Dataset},
  year         = {2026},
  publisher    = {GitHub},
  howpublished = {\url{https://github.com/OSU-NLP-Group/RxnVerif}},
}
```

---

## License

This dataset is released under the [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/) license.
You are free to use, share, and adapt the data for any purpose, provided that appropriate credit is given, a link to the license is included, and any changes made are indicated.
