### Note this tutorial is under construction

This tutorial will demonstrate how to analyze and interpret example 16S sequencing data using our pipeline.

You can download the example data [here](https://www.dropbox.com/s/t0q0p88czo9hk6g/16S_tutorial.tar.gz?dl=0).

This dataset was originally used in a project to determine whether knocking out the protein [chemerin](https://en.wikipedia.org/wiki/Chemerin) affects gut microbial composition. 116 mouse samples acquired from two different facilities were used for this project. Metadata associated with each sample is indicated in the mapping file (map.txt). In this mapping file the genotypes of interest can be seen: wildtype (WT and WT_BZ), chemerin knockout (chemerin_KO), chemerin receptor knockout (CMKLR1_KO) and a heterozygote for the receptor knockout (HET). Also of importance are the two source facilities: "BZ" and "CJS". It is generally a good idea to include as much metadata as possible, since this data can easily be explored later on.

We first run FastQC in order to check the quality of the sequencing runs:

    fastqc -t 4 raw_data/* -o fastqc_out/

In the fastqc_out folder an html file is created for each fastq file. When you open these html files in your browser you can look over a number of quality metrics (a number of the metrics do not give useful results due to the nature of 16S sequencing, read more [here](https://github.com/mlangill/microbiome_helper/wiki/Sequence-QC)). The most informative metric for our purposes is the "Per base sequence quality", which shows the Phred quality score distribution along the reads. Generally these distributions are skewed lower near the 3' read ends. Also, since all reads have the same forward primer sequence there is no variation at the first few positions, as in the below image for 1CMK6WT_S218_L001_R1_001.fastq:

![](https://www.dropbox.com/s/2upj9evprw7a8yv/FastQC_boxplot_example.jpg?raw=1)

By looking at a few of these plots you can catch any major sequencing errors that may have occurred. Since we are dealing with paired-end reads, these quality scores will increase once the reads are stitched together. We will first stitch the reads together before  

