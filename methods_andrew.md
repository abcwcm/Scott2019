Methods for Scott et al., 2019
===============================

>Applied Bioinformatics Core, Weill Cornell Medicine

* [RNA-seq](#rnaseq)
* [ATAC-seq](#attac)
* [References](#refs)

The quality of the sequenced reads was assessed with `FastQC` and `QoRTs` (for RNA-seq samples).
The scripts for preprocessing including alignment, coverage and count file generation and normalization can be found in the folder [`preprocessing`](https://github.com/friedue/Scott2019/blob/master/preprocessing/).
Unless stated otherwise, all plots involving high-throughput sequencing data were obtained with custom R scripts (see the folder [`code_for_figures`](https://github.com/friedue/Scott2019/tree/master/code_for_figures)).

## RNA-seq <a name="rnaseq"></a>

DNA reads were aligned with default parameters to the mouse reference genome (GRCm38) using `STAR`.
Gene expression estimates were obtained with `featureCounts` using composite gene models (union of the exons of all transcript isoforms per gene) from Gencode (version M17).

### Differentially expressed genes

Differentially expressed genes (DEG) were determined with `DESeq2`.
The q-value cut-offs for the final lists of DEG were as follows: 

- TOX-GFP vs. GFP: 849 DEG with q-value smaller than 0.10
- TAG vs. OT1: 2347 DEG with q-value smaller than 0.05
- WT vs. TOX KO: 679 DEG with q-value smaller than 0.05

### Pathway and GO term enrichment analyses

Gene set enrichment analyses were done using GSEA on Reads Per Kilobase Million (RPKM) values against a gene set permutation (the seed was set to 149).   

### Heatmaps
Heatmaps were created using log2 counts per million (CPMs) of genes identified as differentially expressed by `DESeq2` (adjusted p < 0.05 unless otherwise noted). Rows were centered and scaled. 

## ATAC-seq <a name="attac"></a>

ATAC-seq data published by Philip et al, 2017 were downloaded from GEO (series: GSE89308).
These data sets were processed in the same manner as the newly generated data sets described in this study.

### Alignment and identification of open chromatin regions

The data was processed following the recommendations of the ENCODE consortium:
Reads were aligned to the mouse reference genome (version GRCm38) with `BWA-backtrack` .
Post-alignment filtering was done with `samtools` and Picard tools to remove unmapped reads, improperly paired reads, non-unique reads, and duplicates.

To identify regions of open chromatin represented by enrichments of reads, peak calling was performed with `MACS2` .
For every replicate, the `narrowpeak` results of `MACS2` were used after filtering for adjusted p-values smaller than 0.01.

### Differentially accessible regions

Regions where the chromatin accessibility changed between different conditions were identified with `diffBind` : with the following options: `minOverlap=4, bUseSummarizeOverlaps=T, minMembers=2, bFullLibrarySize=TRUE`.

### Coverage files

Individual coverage files per replicate normalized for differences in sequencing depths between the different samples were generated with `bamCoverage` of the deepTools suite using the following parameters: `-bs 10 --normalizeUsing RPGC --effectiveGenomeSize 2150570000 --blackListFileName mm10.blacklist --ignoreForNormalization chrX chrY --ignoreDuplicates --minFragmentLength 40 -p 1`.

To create merged coverage files of replicates of the same condition, we used `multiBigwigSummary` to obtain the sequencing-depth-normalized coverage values for 10 bp bins along the entire genome, i.e. for every condition, we obtained a table with the coverage values in every replicate within the same bin.
Subsequently, we chose the mean value for every bin to represent the coverage in the resulting "merged" file (see [averaging_bigWigs_example.sh](https://github.com/friedue/Scott2019/blob/master/preprocessing/averaging_bigWigs_example.sh)).

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


## References <a name="refs"></a>

Andrews, Simon. 2010. http://www.bioinformatics.babraham.ac.uk/projects/fastqc/.

Benner, Christopher, Sven Heinz, and Christopher K Glass. 2017. “HOMER - Software for motif discovery and next generation sequencing analysis.” http://homer.Ucsd.Edu/.

Broad Institute. 2015. “Picard.” http://broadinstitute.github.io/picard/.

Dobin, Alexander, Carrie A. Davis, Felix Schlesinger, Jorg Drenkow, Chris Zaleski, Sonali Jha, Philippe Batut, Mark Chaisson, and Thomas R. Gingeras. 2013. “STAR: Ultrafast universal RNA-seq aligner.” Bioinformatics 29 (1): 15–21. doi:10.1093/bioinformatics/bts635.

Harrow, Jennifer, Adam Frankish, Jose M Gonzalez, Electra Tapanari, Mark Diekhans, Felix Kokocinski, Bronwen L Aken, et al. 2012. “GENCODE: The reference human genome annotation for the ENCODE Project.” Genome Research 22 (9): 1760–74. doi:10.1101/gr.135350.111.

Hartley, Stephen W., and James C. Mullikin. 2015. “QoRTs: a comprehensive toolset for quality control and data processing of RNA-Seq experiments.” BMC Bioinformatics 16: 224. doi:10.1186/s12859-015-0670-5.

Heinz, Sven, Christopher Benner, Nathanael Spann, Eric Bertolino, Yin C. Lin, Peter Laslo, Jason X. Cheng, Cornelis Murre, Harinder Singh, and Christopher K. Glass. 2010. “Simple Combinations of Lineage-Determining Transcription Factors Prime cis-Regulatory Elements Required for Macrophage and B Cell Identities.” Molecular Cell. doi:10.1016/j.molcel.2010.05.004.

Langmead, B, C Trapnell, M Pop, and S L Salzberg. 2009. “Ultrafast and memory-efficient alignment of short DNA sequences to the human genome.” Genome Biology, 1–10. doi:10.1186/gb-2009-10-3-r25.

Li, H., and R. Durbin. 2009. “Fast and accurate short read alignment with Burrows-Wheeler transform.” Bioinformatics 25: 1754–60. doi:10.1093/bioinformatics/btp324.

Li, Heng, and Richard Durbin. 2009. “Fast and accurate short read alignment with Burrows-Wheeler transform.” Bioinformatics 25 (14): 1754–60. doi:10.1093/bioinformatics/btp324.

Li, Heng, Bob Handsaker, Alec Wysoker, Tim Fennell, Jue Ruan, Nils Homer, Gabor Marth, Goncalo Abecasis, and Richard Durbin. 2009. “The Sequence Alignment/Map format and SAMtools.” Bioinformatics 25 (16): 2078–9. doi:10.1093/bioinformatics/btp352.

Liao, Yang, Gordon K. Smyth, and Wei Shi. 2014. “FeatureCounts: An efficient general purpose program for assigning sequence reads to genomic features.” Bioinformatics 30 (7): 923–30. doi:10.1093/bioinformatics/btt656.

Liu, Tao. 2014. “Use Model-Based Analysis of ChIP-Seq (MACS) to Analyze Short Reads Generated by Sequencing Protein-DNA Interactions in Embryonic Stem Cells.” In Stem Cell Transcriptional Networks: Methods and Protocols, edited by Benjamin L Kidder, 1150:201–12. doi:10.1007/978-1-4939-0512-6.

Martin, Marcel. 2011. “Cutadapt removes adapter sequences from high-throughput sequencing reads.” EMBnet.journal 17 (1): 10–12. doi:http://dx.doi.org/10.14806/ej.17.1.200.

Martinez, Gustavo J., Renata M. Pereira, Tarmo Äijö, Edward Y. Kim, Francesco Marangoni, Matthew E. Pipkin, Susan Togher, et al. 2015. “The Transcription Factor NFAT Promotes Exhaustion of Activated CD8+ T Cells.” Immunity 42 (2): 265–78. doi:10.1016/j.immuni.2015.01.006.

Philip, Mary, Lauren Fairchild, Liping Sun, Ellen L. Horste, Steven Camara, Mojdeh Shakiba, Andrew C. Scott, et al. 2017. “Chromatin states define tumour-specific T cell dysfunction and reprogramming.” Nature 545 (7655): 452–56. doi:10.1038/nature22367.

R Core Team. 2014. R: A Language and Environment for Statistical Computing. Vienna, Austria: R Foundation for Statistical Computing. https://www.R-project.org/.

Ramìrez, Fidel, Devon P Ryan, Bjöorn Grüning, Vivek Bhardwaj, Fabian Kilpert, Andreas S Richter, Steffen Heyne, Friederike Dündar, and Thomas Manke. 2016. “deepTools2: a next generation web server for deep-sequencing data analysis.” Nucleic Acids Research 44 (April): 160–65. doi:10.1093/nar/gkw257.

Stark, R, and G Brown. 2011. DiffBind: Differential Binding Analysis of Chip-Seq Peak Data. http://bioconductor.org/packages/release/bioc/vignettes/DiffBind/inst/doc/DiffBind.pdf.

Subramanian, Aravind, Pablo Tamayo, Vamsi K Mootha, Sayan Mukherjee, Benjamin L Ebert, Michael A Gillette, Amanda Paulovich, et al. 2005. “Gene set enrichment analysis: a knowledge-based approach for interpreting genome-wide expression profiles.” PNAS 102 (43): 15545–50. doi:10.1073/pnas.0506580102.

The ENCODE Consortium. “ATAC-seq Data Standards.” https://www.encodeproject.org/data-standards/atac-seq/.

Wickham, Hadley. 2009. Ggplot2: Elegant Graphics for Data Analysis. Springer New York.

