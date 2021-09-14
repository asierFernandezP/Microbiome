# QIIME2 analysis workflow
<br> 

## Introduction

This file summarizes the pipeline for the analysis of 16s rRNA Amplicon Sequencing Data starting with the FASTQ files with removed barcodes and primer sequences. For this, QIIME2 tool is used. 

## Content

We here describe the methods used to perform the next steps: 

1. Data import and quality checking of the reads.
2. ASV determination with DADA2.
3. Alignment of the representative sequences, building a phylogenetic tree and taxonomic assignment.
<br> 

First, you need to activate QIIME2:


```
conda activate qiime2-2020.2
```
<br> 

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

Next, you can visualize the quality of the reads and check if it is necessary to trim the sequences. For this you can use the following command or simply drop the qzv file in the browser ([QIIME 2 View](https://view.qiime2.org/)):

```
qiime tools view paired-end-demux.qzv
```

You can generate the metadata file. It is a tabulated text file, in which the first line indicates the different fields (name of sample, mouse models, treatment...). The first column (#SampleID) always corresponds to the sample name (same name used in the samplemanifest file).

<br>

### 2) ASV determination with DADA2

This can be done in R following [DADA2 pipeline](https://benjjneb.github.io/dada2/tutorial.html), but QIIME2 offers also this analysis in a single command:

```
qiime dada2 denoise-paired  --i-demultiplexed-seqs paired-end-demux.qza  --p-trim-left-f 0  --p-trunc-len-f 0  --p-trim-left-r 0  --p-trunc-len-r 0  --o-representative-sequences rep-seqs.qza  --o-table table.qza  --o-denoising-stats stats.qza  --p-n-threads 2  --p-n-reads-learn 1505813
```

In this case, 0 is indicated as the position to trim (therefore, no trimming) both in the 5' and 3' of the forward and reverse reads. Finally, the 25% of the total reads are used to train the error model (1505813 in this case). 

Now you can visualize the resulting artifacts:

```
qiime metadata tabulate \
 --m-input-file stats.qza \
 --o-visualization stats.qzv

qiime feature-table summarize \
 --i-table table.qza \
 --o-visualization table.qzv \
 --m-sample-metadata-file metadata.tsv
 
qiime feature-table tabulate-seqs \
 --i-data rep-seqs.qza \
 --o-visualization rep-seqs.qzv
```

After this step, you can remove singletons and ASVs with low frequency. For this, a good approach is to remove all ASVs with a lower frequency than 0.1% of the mean depth. The mean depth in this analysis is 103245 (as indicated when visualizing table.qzv). The 0.1% of this number is approximately 103. Then, you can use use the following command:

```
qiime feature-table filter-features --i-table table.qza \
 --p-min-frequency 103 \
 --p-min-samples 1 \
 --o-filtered-table table_filt.qza 
```

You can also visualize the summary of the new filtered table:

```
qiime feature-table summarize \
 --i-table table_filt.qza \
 --o-visualization table_filt.qzv
 ```

You may want to filter these ASVs also from the file containing the representative sequences and visualize it:

```
qiime feature-table filter-seqs --i-data rep-seqs.qza \
 --i-table table_filt.qza \
 --o-filtered-data rep-seqs_filt.qza

qiime feature-table tabulate-seqs \
 --i-data rep-seqs_filt.qza \
 --o-visualization rep-seqs_filt.qzv
```

<br> 

### 3) Alignment of the representative sequences, building a phylogenetic tree and taxonomic assignment
