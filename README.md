# 16S analysis workflow
<br> 


## Introduction

This file summarizes the pipeline for the analysis of 16s rRNA Amplicon Sequencing Data starting with the FASTQ files with removed barcodes and primer sequences.

## Content

We here describe the methods used to perform the next steps: 

1. Quality checking of the reads.
2. ASV determination with DADA2.
3. Alignment of the representative sequences, building a phylogenetic tree and taxonomic assignment.
4. Statistical analysis.


The pipeline for these analyses are summarized in the next files:

a) Bash scripts:

- [QIIME2 analysis](Bash/QIIME2_analysis.md) (1,2,3)

b) R scripts:

- [Statistical analysis](R/Descriptive_data_analysis.R) (4)
 <br> <br> <br>

The project workflow is summarized in the next figure:
![GitHub Logo](images/QIIME2_overview.png)

<sup>**Figure 1. Workflow of the 16S analysis pipeline.**  </sup>
