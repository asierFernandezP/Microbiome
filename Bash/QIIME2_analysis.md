# QIIME2 analysis workflow
<br> 

## Introduction

This file summarizes the pipeline for the analysis of 16s rRNA Amplicon Sequencing Data starting with the FASTQ files with removed barcodes and primer sequences. For this, QIIME2 tool is used. 

## Content

We here describe the methods used to perform the next steps: 

1. Quality checking of the reads.
2. ASV determination with DADA2.
3. Alignment of the representative sequences, building a phylogenetic tree and taxonomic assignment.
<br> 

First, we have to activate QIIME2:


```
conda activate qiime2-2020.2
```



### 1) Data import and quality checking 

First, you have to create the samplesmanifest file with the location of the different FASTQ files.

Next, you can import the FASTQ files with the following command:

```
qiime tools import --type 'SampleData[PairedEndSequencesWithQuality]'  --input-path samplemanifest  --output-path paired-end-demux.qza  --input-format PairedEndFastqManifestPhred33V2
```

Files with extension "qza" are artefacts. We cannot visualize them. For this, we use the following command to convert it into a visualization file ("qzv" extension):

```
qiime demux summarize --i-data paired-end-demux.qza --o-visualization paired-end-demux.qzv
```

Next, you can visualize the quality of the reads and check if it is necessary to trim the sequences. For this you can use the following command or simply drop the qzv file in the browser (https://view.qiime2.org/):

```
qiime tools view paired-end-demux.qzv
```

You can generate the metadata file. It is a tabulated text file, in which the first line indicates the different fields (name of sample, mouse models, treatment...). The first column (#SampleID) always corresponds to the sample name (same name used in the samplemanifest file).

### 2) ASV determination with DADA2


### 3) Alignment of the representative sequences, building a phylogenetic tree and taxonomic assignment
