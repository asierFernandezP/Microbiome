# QIIME2 analysis workflow
<br> 

## Introduction

This file summarizes the pipeline for the analysis of 16S rRNA Amplicon Sequencing Data starting with the FASTQ files with removed barcodes and primer sequences. For this, QIIME2 tool is used. 

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

You can generate the metadata file. It is a tabulated text file, in which the first row indicates the different fields (name of sample, mouse models, treatment...). The first column (#SampleID) always corresponds to the sample name (same name used in the samplemanifest file).

<br>

### 2) ASV determination with DADA2

This can be done in R following [DADA2 pipeline](https://benjjneb.github.io/dada2/tutorial.html), but QIIME2 offers also this analysis in a single command:

```
qiime dada2 denoise-paired  --i-demultiplexed-seqs paired-end-demux.qza  --p-trim-left-f 0  --p-trunc-len-f 0  --p-trim-left-r 0  --p-trunc-len-r 0  --o-representative-sequences rep-seqs.qza  --o-table table.qza  --o-denoising-stats stats.qza  --p-n-threads 2  --p-n-reads-learn 1505813
```

In this case, 0 is indicated as the position to trim (therefore, no trimming) both in the 5' and 3' of the forward and reverse reads. Finally, the 25% of the total reads are used to train the error model (1,505,813 in this case). 

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

After this step, you can remove singletons and ASVs with low frequency. For this, a good approach is to remove all ASVs with a lower frequency than 0.1% of the mean depth. The mean depth in this analysis is 103,245 (as indicated when visualizing table.qzv). The 0.1% of this number is approximately 103. Then, you can use use the following command:

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

You can now align the representative sequences using the MAFFT algorithm and use FastTree to build the phylogenetic tree:

```
qiime phylogeny align-to-tree-mafft-fasttree \
 --i-sequences rep-seqs_filt.qza \
 --o-alignment aligned-rep-seqs.qza \
 --o-masked-alignment masked-aligned-rep-seqs.qza \
 --o-tree unrooted-tree.qza \
 --o-rooted-tree rooted-tree.qza
 
```
Once you have aligned the sequences and obtained the philogeny, the next step would be the taxonomic assignment of each ASV. For this, you need to perform several steps:
<br> 

**A) Generation of the database for the taxonomy assignment:** in this case, a recomenndable option would be the SILVA database 99%. For this, we can download the artifacts with the sequences (full-length) and the taxonomic assigments from [QIIME2 data resources](https://docs.qiime2.org/2020.8/data-resources/).

**B) Extract the reference reads:** the primers here used are designed to amplify the 515-806 region of the 16S rRNA gene. There are already available and trained classifiers that can be downloded from [QIIME2 data resources](https://docs.qiime2.org/2020.8/data-resources/). However, in general, for 16S rRNA it is recommended to train you own classifier based on your specific sample preparation and sequencing parameters (including, for example, only the region of the target sequences that was sequenced):

```
qiime feature-classifier extract-reads \
  --i-sequences silva-138-99-seqs.qza \
  --p-f-primer GTGCCAGCMGCCGCGGTAA \
  --p-r-primer GGACTACHVGGGTWTCTAAT \
  --p-min-length 100 \
  --p-max-length 400 \
  --o-reads reference-seqs.qza
```

**C) Training the classifier:** now you need to train the Naive-Bayes classifier, associating the sequences for the database that have been trimmed with their taxonomy:

```
qiime feature-classifier fit-classifier-naive-bayes \
  --i-reference-reads reference-seqs.qza \
  --i-reference-taxonomy silva-138-99-tax.qza \
  --o-classifier classifier.qza
```

**D) Taxonomic assigment of the representative sequences of each ASV:**

```
qiime feature-classifier classify-sklearn \
  --i-classifier classifier.qza \ 
  --i-reads rep-seqs_filt.qza \
  --p-pre-dispatch 1 \
  --o-classification taxonomy.qza
  ```
You can now visualize the results of the taxonomic assignment:

```
qiime metadata tabulate \
  --m-input-file taxonomy.qza \
  --o-visualization taxonomy.qzv
```

You can also generate a plot with the abundance of each ASV:

```
qiime taxa barplot --i-table table_filt.qza  \
 --i-taxonomy taxonomy.qza \
 --m-metadata-file  metadata.tsv \
 --o-visualization taxa_barplot.qzv
```
