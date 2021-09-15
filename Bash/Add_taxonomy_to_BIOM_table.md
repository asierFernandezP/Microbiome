In this file you can find the pipeline to generate a table with the abundance of each ASV in each sample and their taxonomy. For this we follow the steps summarized in [QIIME2 forum](https://forum.qiime2.org/t/exporting-and-modifying-biom-tables-e-g-adding-taxonomy-annotations/3630):


First, you need to export the BIOM table (feature table) and the taxonomy:

```
mkdir export
qiime tools export --input-path table_filt.qza --output-path exported
qiime tools export --input-path taxonomy.qza --output-path exported
```

Next, you will need to modify the exported taxonomy file’s header before using it with BIOM software. Before modifying that file, make a copy:

```
cd exported
cp taxonomy.tsv biom-taxonomy.tsv
```

Change the first line of biom-taxonomy.tsv (i.e. the header) to the following one (separated by tab):

```
#OTUID  taxonomy  confidence
```

Finally, yo need to add the taxonomy data to your .biom file and convert the .biom file to .tsv:

```
biom add-metadata -i feature-table.biom -o table-with-taxonomy.biom --observation-metadata-fp biom-taxonomy.tsv --sc-separated taxonomy
biom convert -i table-with-taxonomy.biom -o table-with-taxonomy-from_biom.tsv --to-tsv --header-key taxonomy 
```
