Methods for Scott et al., 2019
===============================

>Applied Bioinformatics Core, Weill Cornell Medicine

## Bioinformatics Methods

The quality of the sequenced reads was assessed with `FastQC` and `QoRTs` (for RNA-seq samples).
Unless stated otherwise, all plots involving high-throughput sequencing data were obtained with custom R scripts (see github.com/friedue/Scott2019 for the code).

### RNA-seq

DNA reads were aligned with default parameters to the mouse reference genome (GRCm38) using `STAR`.
Gene expression estimates were obtained with `featureCounts` using composite gene models (union of the exons of all transcript isoforms per gene) from Gencode (version M17).

#### Differentially expressed genes

Differentially expressed genes (DEG) were determined with `DESeq2`.
The q-value cut-offs for the final lists of DEG were as follows: 

- TOX-GFP vs. GFP: 849 DEG with q-value smaller than 0.10
- TAG vs. OT1: 2347 DEG with q-value smaller than 0.05
- WT vs. TOX KO: 679 DEG with q-value smaller than 0.05

#### Pathway and GO term enrichment analyses

Gene set enrichment analyses were done using GSEA on Reads Per Kilobase Million (RPKM) values against a gene set permutation (the seed was set to 149).   

#### Heatmaps
Heatmaps were created using log2 counts per million (CPMs) of genes identified as differentially expressed by `DESeq2` (adjusted p < 0.05 unless otherwise noted). Rows were centered and scaled. 

### ATAC-seq

ATAC-seq data published by Philip et al, 2017 were downloaded from GEO (series: GSE89308).
These data sets were processed in the same manner as the newly generated data sets described in this study.

#### Alignment and identification of open chromatin regions

The data was processed following the recommendations of the ENCODE consortium:
Reads were aligned to the mouse reference genome (version GRCm38) with `BWA-backtrack` .
Post-alignment filtering was done with `samtools` and Picard tools to remove unmapped reads, improperly paired reads, non-unique reads, and duplicates.

To identify regions of open chromatin represented by enrichments of reads, peak calling was performed with `MACS2` .
For every replicate, the `narrowpeak` results of `MACS2` were used after filtering for adjusted p-values smaller than 0.01.

#### Differentially accessible regions

Regions where the chromatin accessibility changed between different conditions were identified with `diffBind` : with the following options: `minOverlap=4, bUseSummarizeOverlaps=T, minMembers=2, bFullLibrarySize=TRUE`.

#### Coverage files

Individual coverage files per replicate normalized for differences in sequencing depths between the different samples were generated with `bamCoverage` of the deepTools suite using the following parameters: `-bs 10 --normalizeUsing RPGC --effectiveGenomeSize 2150570000 --blackListFileName mm10.blacklist --ignoreForNormalization chrX chrY --ignoreDuplicates --minFragmentLength 40 -p 1`.

To create merged coverage files of replicates of the same condition, we used `multiBigwigSummary` to obtain the sequencing-depth-normalized coverage values for 10 bp bins along the entire genome, i.e. for every condition, we obtained a table with the coverage values in every replicate within the same bin.
Subsequently, we chose the mean value for every bin to represent the coverage in the resulting "merged" file (see github.com/friedue/Scott2019 for the actual code that was used).

Merged coverage files were used for display in IGV (e.g. Fig 2i, 5g) and for heatmaps shown in Figures 2h and 5e.

#### Heatmaps

Heatmaps displaying the sequencing-depth-normalized coverage from different ATAC-seq samples as shown in Fig. 2h and 5e were generated with `computeMatrix` and `plotHeatmap` of the deepTools suite.

Every row corresponds to a single region that was determined to be differentially accessible when comparing either TAG to OT1 T cells (Fig. 2h) or WT to TOX KO T cells (Fig. 5e).
The plots display the center of each differentially accessible peak region +/- 1kb; the color corresponds to the average normalized coverage across all replicates of the respective condition.
Gene labels indicate genes that overlapped with a given differentially accessible region (anywhere along the gene).

#### Motif analyses
Motif analysis was then run separately on hyper- or hypo-accessible peaks in each comparison using `HOMER` v-4.9.1, with the flags `-size given -mask`.  Motifs that fell below P<0.05 in both lists were removed.  Motifs enriched in hyper- or hypo- accessible peaks were obtained by taking the rank difference of the motifs in the two lists. Top differentially ranked motifs were plotted in a barplot representing their P-value of enrichment.


### Combining RNA-seq and ATAC-seq data
The relationship between RNA-seq and ATAC-seq was explorered via "diamond" plots for select genes detected as differentially expressed via `DESeq2`. Each gene was represented by a stack of diamond-shaped points colored by that gene’s associated chromatin state (blue indicating closing and red indicating opening). The bottom-most point in each stack corresponds to the log2 fold change in expression of that gene. 


### ChIP-seq

#### NFAT1 ChIP-seq (publicly available)

NFAT1-ChIP-seq samples were generated by Martinez et al., 2015 from cells expressing endogenous NFAT1 ("WT") or lacking NFAT1 ("KO").
Cells lacking endogenous NFAT1 were transduced with an empty GFP vector ("Mock") or with a vector containing a mutated form of NFAT ("CA-RIT-RV").
Either cell type was either left resting ("None") or stimulated with PMA and ionomycin ("P+I") for 1 hour.

We downloaded the sequencing results (fastq files generated by SOLiD sequencing technology) from the Sequence Read Archive (GEO series GSE64407).
SOLiD adapters had to be trimmed off, which we did with `cutadapt` specifying `--format=sra-fastq  --minimum-length 15 --colorspace` and the sample specific adapter sequences via `-g` and `-a` (see https://ars.els-cdn.com/content/image/1-s2.0-S1074761315000321-mmc6.xlsx for the sample-specific adapters).
The trimmed reads were subsequently aligned to the mouse genome version GRCm38 with `bowtie1` using the colorspace option.

Coverage tracks normalized for differences in sequencing depths were be generated with `bamCoverage` of the `deepTools` suite (v3.1.0) using the following parameters: 
`-bs 10 --normalizeUsing RPGC --effectiveGenomeSize 2150570000 --blackListFileName mm10.blacklist --ignoreForNormalization chrX chrY --ignoreDuplicates --minFragmentLength 40 -p 1`.

Blacklisted regions were downloaded from https://sites.google.com/site/anshulkundaje/projects/blacklists.

Regions of statistically significant read enrichments in the ChIP samples compared to the corresponding input samples ("peaks") were identified with `MACS2` (2.1.1.20160309) using ChIP and corresponding input files and the following parameters: `-g 1.87e9 -p 0.01 --keep-dup all`.
For final peak files, the `narrowpeak` outputs of `MACS2` were used, keeping only peaks with adjusted p-values below 0.01.

| SRA Run | GEO  | GEO sample name						| Our sample name		|
|--------|-----------------|---------------------------|------------------|
|	SRR1731133 | GSM1570757 |  NFAT1 KO Mock None NFAT1 IP   | KO-NFAT_noStim_ChIP |
|	SRR1731127 | GSM1570751 | NFAT1 KO Mock None input       | KO-NFAT_noStim_input |
| SRR1731136 | GSM1570760 | NFAT1 KO Mock P+I 1h NFAT1 IP  | KO-NFAT_Stim_ChIP   |
| SRR1731130 | GSM1570754 | NFAT1 KO Mock P+I 1h input     | KO-NFAT_Stim_input   |
|	SRR1731129 | GSM1570753 | WT Mock P+I 1h | input         | WT-NFAT_Stim_input |
| SRR1731135 | GSM1570759 | WT Mock P+I 1h NFAT1 IP        | WT-NFAT_Stim_ChIP |
| SRR1731134 | GSM1570758 | NFAT1 KO CA-RIT-RV None NFAT1 IP   | MUT-NFAT_noStim_ChIP  |
| SRR1731128 | GSM1570752 | NFAT1 KO CA-RIT-RV None input      |MUT-NFAT_noStim_input |
| SRR1731137 | GSM1570761 | NFAT1 KO CA-RIT-RV P+I 1h NFAT1 IP  | MUT-NFAT_Stim_ChIP  |
| SRR1731131 | GSM1570755 | NFAT1 KO CA-RIT-RV P+I 1h input     | MUT-NFAT_Stim_input |


#### TOX ChIP-seq

Processing was performed by Active Motif.
In short, reads were aligned to the mm10 genome version using the BWA algorithm with default settings.
Reads that aligned with no more than 2 mismatches and mapped uniquely to the genome were used in the subsequent analysis; in addition, duplicate reads were removed.

Coverage files (bigWig) were obtained by extending the reads in silico (using Active Motif software) at their 3' ends to a length of 150-250 bp, depending on the average fragment length in the size selected library. To identify the density of fragments (extended tags) along the genome, the genome is divided into 32-nt bins and the number of fragments in each bin is determined (histogram of fragment densities).

Peaks were identified with `MACS2` (v2.1.0) with default cutoff (p-value 1e-7) for narrow peaks. Peak filtering was performed by removing false ChIP-Seq peaks as defined within the ENCODE blacklist.

De novo motifs were identified with the `findMotifsGenome` program of the HOMER package using default parameters and input sequences comprising +/- 100 bp from the center of the top 1,000 peaks.

To assess the overlap of peaks with annotated genes, gene annotations defined by RefSeq(GRCm38.p1-C57BL/6J) were used.
An overlap was defined as a minimal 1-bp overlap between the MACS2 summit interval files (which are 1 bp intervals) and the feature annotation interval.
