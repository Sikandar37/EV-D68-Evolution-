# EV-D68 Evolutionary Analysis Pipeline

This repository contains the complete workflow used to reconstruct the evolutionary history of **Enterovirus D68 (EV-D68)** using maximum likelihood phylogenetic inference and the Nextstrain Augur pipeline.

The pipeline includes sequence alignment, model selection, phylogenetic reconstruction, temporal calibration, ancestral state reconstruction, mutation inference, and interactive visualization using Auspice.

---

# Repository Contents

All input, configuration, and output files are stored in a single working directory.

```
EV-D68-Evolution/
│
├── EVD68_alignment.fasta
├── metadata.csv
├── reference.gb
├── colors.tsv
├── auspice_config.json
│
├── EVD68.treefile
├── tree.nwk
├── branch_lengths.json
├── traits.json
├── nt_muts.json
├── aa_muts.json
├── EVD68.json
│
└── README.md
```

---

# Input Data

The following input files are required to run the workflow:

| File | Description |
|------|-------------|
| `EVD68_alignment.fasta` | Multiple sequence alignment of EV-D68 nucleotide sequences |
| `metadata.csv` | Sample metadata including collection date, country, region, and clade membership |
| `reference.gb` | Annotated EV-D68 reference genome used for mutation annotation |
| `colors.tsv` | Trait color definitions for Auspice visualization |
| `auspice_config.json` | Configuration file for Auspice output |

---

# Metadata Format

The metadata file (`metadata.csv`) must contain one row per sequence.

Required columns:

| Column | Description |
|--------|-------------|
| strain | Sequence identifier matching FASTA headers |
| date | Collection date (YYYY-MM-DD or YYYY) |
| country | Country of sample origin |
| region | Broad geographic region |
| clade_membership | Assigned EV-D68 clade |

FASTA headers must match the `strain` column exactly.

---

# Workflow Overview

## 1. Model Selection
The best-fitting nucleotide substitution model was identified using IQ-TREE ModelFinder.

```bash
iqtree2 -s EVD68_alignment.fasta -m TESTONLY
```
The optimal substitution model was selected according to the Bayesian Information Criterion (BIC) and Akaike Information Criterion (AIC).

---

## 2. Maximum Likelihood Tree Reconstruction
The maximum likelihood phylogeny was inferred using IQ-TREE with 1,000 ultrafast bootstrap replicates.
```bash
iqtree2 -s EVD68_alignment.fasta -m GTR+F+I+G4 -bb 1000 -nt AUTO
```

The resulting tree file (`EVD68_alignment.fasta.treefile`) was used as the starting tree for temporal phylogenetic reconstruction in Nextstrain..

---

## 3. Time-Calibrated Phylogeny
Activate the Nextstrain environment:
```bash
nextstrain shell .

Estimate the molecular clock and generate the time-scaled phylogeny.

```bash
augur refine \
    --tree EVD68.treefile \
    --alignment EVD68_alignment.fasta \
    --metadata metadata.csv \
    --output-tree tree.nwk \
    --output-node-data branch_lengths.json \
    --timetree \
    --coalescent opt \
    --date-confidence \
    --date-inference marginal \
    --clock-filter-iqd 4
```

---

## 4. Trait Reconstruction
Infer ancestral geographic traits and clade memberships.
```bash
augur traits \
    --tree tree.nwk \
    --metadata metadata.csv \
    --output-node-data traits.json \
    --columns region country clade_membership
```

---

## 5. Ancestral Sequence Reconstruction
Infer ancestral nucleotide sequences.
```bash
augur ancestral \
    --tree tree.nwk \
    --alignment EVD68_alignment.fasta \
    --output-node-data nt_muts.json \
    --inference joint
```
This step identifies nucleotide substitutions occurring along each branch of the phylogeny

---

## 6. Amino Acid Mutation Inference
Translate nucleotide substitutions into amino acid mutations.
```bash
augur translate \
    --tree tree.nwk \
    --ancestral-sequences nt_muts.json \
    --reference-sequence reference.gb \
    --output-node-data aa_muts.json
```

---

## 7. Export for Auspice
Combine all inferred annotations into an Auspice-compatible JSON file.
```bash
augur export v2 \
    --tree tree.nwk \
    --metadata metadata.csv \
    --node-data branch_lengths.json traits.json nt_muts.json aa_muts.json \
    --colors colors.tsv \
    --auspice-config auspice_config.json \
    --output EVD68.json
```

---

## 8. Visualization
Open the Auspice web interface and upload the generated **EVD68.json** file to visualize:

- Open https://auspice.us
- Upload `EVD68.json`
- Explore phylogeny, temporal dynamics, geographic spread, and mutations

---
# 9. Workflow Summary

```
Genome sequences
       │
       ▼
Multiple sequence alignment
       │
       ▼
IQ-TREE Model Selection
       │
       ▼
Maximum Likelihood Tree
       │
       ▼
Augur Refine
(Time-calibrated tree)
       │
       ├─────────────┐
       ▼             ▼
Trait inference   Ancestral sequences
       │             │
       └──────┬──────┘
              ▼
Amino acid mutations
              │
              ▼
Augur Export (JSON)
              │
              ▼
Auspice Visualization
```

---


# Code Availability

All scripts and commands used for EV-D68 time-scaled phylogenetic analysis are provided in this repository. 

---
