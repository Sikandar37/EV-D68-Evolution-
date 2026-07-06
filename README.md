# EV-D68-Evolution- 
This repository contains the workflow used to reconstruct the evolutionary history of Enterovirus D68 (EV-D68) using maximum likelihood phylogenetic inference and the Nextstrain Augur pipeline. The workflow performs sequence alignment, phylogenetic reconstruction, temporal calibration, ancestral state reconstruction, amino acid mutation inference, and interactive visualization with Auspice.
EV-D68-Nextstrain/
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
├── auspice/
│
├── scripts/
│
└── README.md
# 1. Metadata Preparation

The metadata file (metadata.csv) contains epidemiological information associated with each viral genome.

Additional metadata fields were included:

country – extracted from the sequence headers.
region – countries grouped into broader geographical regions (e.g., Asia, Europe, Africa, North America, South America, Oceania).
clade_membership – assigned according to the study-specific clade classification.

Example metadata columns:
| strain    | date       | country | region        | clade_membership |
| --------- | ---------- | ------- | ------------- | ---------------- |
| EVD68_001 | 2024-05-12 | Japan   | Asia          | B3               |
| EVD68_002 | 2023-11-08 | USA     | North America | A2               |
# 2.  Maximum Likelihood Phylogenetic Analysis
1. Model Selection
The optimal nucleotide substitution model was identified using IQ-TREE ModelFinder.
iqtree2 -s** EVD68_alignment.fasta **-m TESTONLY
# 3. Maximum Likelihood Tree Reconstruction
The maximum likelihood phylogeny was inferred using IQ-TREE with 1,000 ultrafast bootstrap replicates
iqtree2 \
    -s **EVD68_alignment.fasta** -m model-name -bb 1000 -nt AUTO
The resulting tree (**EVD68_alignment.fasta.treefile**) was used as the starting tree for temporal phylogenetic reconstruction in Nextstrain.
# 4. Time-Calibrated Phylogeny
Activate the Nextstrain environment:
nextstrain shell .
Run Augur refine to estimate a molecular clock and construct a time-scaled phylogeny.
augur refine --tree **EVD68.treefile** --alignment **EVD68-alignment.fasta**  --metadata **metadata.csv** --output-tree **tree.nwk**  --output-node-data branch_lengths.json --timetree --coalescent opt --date-confidence --date-inference marginal --clock-filter-iqd 4
# 5. Trait Reconstruction
Discrete geographic and clade traits were reconstructed across the phylogeny.
augur traits --tree **tree.nwk** --metadata **metadata.csv** --output-node-data traits.json --columns region country clade_membership
# 6. Ancestral Sequence Reconstruction
Ancestral nucleotide sequences were inferred using joint maximum likelihood reconstruction
augur ancestral --tree **tree.nwk** --alignment **EVD68_alignment.fasta** --output-node-data nt_muts.json --inference joint
This step identifies nucleotide substitutions occurring along each branch of the phylogeny.
# 7. Amino Acid Mutation Inference
Nucleotide substitutions were translated into amino acid changes using the annotated EV-D68 reference genome.
augur translate --tree **tree.nwk** --ancestral-sequences nt_muts.json --reference-sequence **reference.gb** --output-node-data aa_muts.json
# 8. Export for Auspice
All phylogenetic annotations were combined into a single Auspice-compatible JSON file.
augur export v2 --tree tree.nwk --metadata metadata.csv --node-data results branch_lengths.json traits.json nt_muts.json aa_muts.json --colors colors.tsv --auspice-config auspice_config.json --output auspice/EVD68.json
# 9. Interactive Visualization
The exported JSON file can be visualized using Auspice.
open the Auspice web interface _Auspice.us_ and upload the generated EV-D68.json file to explore the time-resolved phylogeny, geographic spread, inferred ancestral traits, and nucleotide and amino acid substitutions.
# 10. Workflow Summary
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
        ├──────────────┐
        ▼              ▼
Trait inference   Ancestral sequences
        │              │
        └──────┬───────┘
               ▼
      Amino acid mutations
               │
               ▼
      Augur Export (JSON)
               │
               ▼
      Auspice Visualization
