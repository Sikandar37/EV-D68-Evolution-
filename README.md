# EV-D68-Evolution

This repository contains the workflow used to reconstruct the evolutionary history of **Enterovirus D68 (EV-D68)** using maximum likelihood phylogenetic inference and the Nextstrain Augur pipeline.

The workflow includes:

- Sequence alignment
- Model selection
- Maximum likelihood phylogenetic reconstruction
- Time-calibrated phylogenetic analysis
- Ancestral trait reconstruction
- Nucleotide and amino acid mutation inference
- Interactive visualization with Auspice

---

# Repository Structure

```
EV-D68-Evolution/
│
├── data/
│   ├── metadata.csv
│   └── sequences.fasta
│
├── results/
│
├── config/
│   ├── colors.tsv
│   ├── lat_longs.tsv
│   ├── auspice_config.json
│   └── reference.gb
│
├── scripts/
│
├── auspice/
│
└── README.md
```

---

# 1. Metadata Preparation

The metadata file (`metadata.csv`) contains epidemiological information associated with each EV-D68 genome.

Additional metadata fields include:

- **country** – extracted from the sequence headers.
- **region** – countries grouped into larger geographical regions (Asia, Europe, Africa, North America, South America, and Oceania).
- **clade_membership** – assigned according to the study-specific clade classification.

Example metadata:

| strain | date | country | region | clade_membership |
|--------|------------|---------|----------------|-----------------|
| EVD68_001 | 2024-05-12 | Japan | Asia | B3 |
| EVD68_002 | 2023-11-08 | USA | North America | A2 |

---

# 2. Maximum Likelihood Phylogenetic Analysis

## Model Selection

The best-fitting nucleotide substitution model was identified using IQ-TREE ModelFinder.

```bash
iqtree2 -s EVD68_alignment.fasta -m TESTONLY
```

The optimal substitution model was selected according to the Bayesian Information Criterion (BIC) and Akaike Information Criterion (AIC).

---

## Maximum Likelihood Tree Reconstruction

The maximum likelihood phylogeny was inferred using IQ-TREE with 1,000 ultrafast bootstrap replicates.

```bash
iqtree2 -s EVD68_alignment.fasta -m GTR+F+I+G4 -bb 1000 -nt AUTO
```

The resulting tree file (`EVD68_alignment.fasta.treefile`) was used as the starting tree for temporal phylogenetic reconstruction in Nextstrain.

---

# 3. Time-Calibrated Phylogeny

Activate the Nextstrain environment:

```bash
nextstrain shell .
```

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

# 4. Trait Reconstruction

Infer ancestral geographic traits and clade memberships.

```bash
augur traits \
    --tree tree.nwk \
    --metadata metadata.csv \
    --output-node-data traits.json \
    --columns region country clade_membership
```

---

# 5. Ancestral Sequence Reconstruction

Infer ancestral nucleotide sequences.

```bash
augur ancestral \
    --tree tree.nwk \
    --alignment EVD68_alignment.fasta \
    --output-node-data nt_muts.json \
    --inference joint
```

This step identifies nucleotide substitutions occurring along each branch of the phylogeny.

---

# 6. Amino Acid Mutation Inference

Translate nucleotide substitutions into amino acid mutations.

```bash
augur translate \
    --tree tree.nwk \
    --ancestral-sequences nt_muts.json \
    --reference-sequence reference.gb \
    --output-node-data aa_muts.json
```

---

# 7. Export for Auspice

Combine all inferred annotations into an Auspice-compatible JSON file.

```bash
augur export v2 \
    --tree tree.nwk \
    --metadata metadata.csv \
    --node-data branch_lengths.json traits.json nt_muts.json aa_muts.json \
    --colors colors.tsv \
    --auspice-config auspice_config.json \
    --output auspice/EVD68.json
```

---

# 8. Interactive Visualization

Open the Auspice web interface and upload the generated **EVD68.json** file to visualize:

- Time-resolved phylogeny
- Geographic spread
- Ancestral trait reconstruction
- Nucleotide mutations
- Amino acid substitutions

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

The complete workflow used for EV-D68 phylogenetic reconstruction, temporal analysis, ancestral state reconstruction, mutation inference, and Auspice visualization is available in this repository.
