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
├── lat_longs.tsv
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
| `lat_longs.tsv` | Geographic coordinates for visualization |
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

```bash
iqtree2 -s EVD68_alignment.fasta -m TESTONLY
```

---

## 2. Maximum Likelihood Tree Reconstruction

```bash
iqtree2 -s EVD68_alignment.fasta -m GTR+F+I+G4 -bb 1000 -nt AUTO
```

The resulting tree file (`EVD68.treefile`) is used for downstream analyses.

---

## 3. Time-Calibrated Phylogeny

```bash
nextstrain shell .

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

```bash
augur traits \
    --tree tree.nwk \
    --metadata metadata.csv \
    --output-node-data traits.json \
    --columns region country clade_membership
```

---

## 5. Ancestral Sequence Reconstruction

```bash
augur ancestral \
    --tree tree.nwk \
    --alignment EVD68_alignment.fasta \
    --output-node-data nt_muts.json \
    --inference joint
```

---

## 6. Amino Acid Mutation Inference

```bash
augur translate \
    --tree tree.nwk \
    --ancestral-sequences nt_muts.json \
    --reference-sequence reference.gb \
    --output-node-data aa_muts.json
```

---

## 7. Export for Auspice

```bash
augur export v2 \
    --tree tree.nwk \
    --metadata metadata.csv \
    --node-data branch_lengths.json traits.json nt_muts.json aa_muts.json \
    --colors colors.tsv \
    --lat-longs lat_longs.tsv \
    --auspice-config auspice_config.json \
    --output EVD68.json
```

---

## 8. Visualization

The resulting `EVD68.json` file can be visualized using Auspice:

- Open https://auspice.us
- Upload `EVD68.json`
- Explore phylogeny, temporal dynamics, geographic spread, and mutations

---

# Workflow Summary

```
Sequences → Alignment → IQ-TREE → Time Tree → Traits → Ancestral Reconstruction
→ Amino Acid Changes → Auspice Export → Visualization
```

---

# Code Availability

All scripts and commands used for EV-D68 phylogenetic and evolutionary analysis are provided in this repository. The workflow is fully reproducible using the input files included here.

---
