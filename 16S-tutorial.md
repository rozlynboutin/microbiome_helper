### Note this tutorial is under construction

This tutorial will demonstrate how to analyze and interpret example 16S sequencing data using our pipeline.

You can download the example data [here](https://www.dropbox.com/s/t0q0p88czo9hk6g/16S_tutorial.tar.gz?dl=0).

This dataset was originally used in a project to determine whether knocking out the protein [chemerin](https://en.wikipedia.org/wiki/Chemerin) affects gut microbial composition. 116 mouse samples acquired from two different facilities were used for this project. Metadata associated with each sample is indicated in the mapping file (map.txt). In this mapping file the genotypes of interest can be seen: wildtype (WT and WT_BZ), chemerin knockout (chemerin_KO), chemerin receptor knockout (CMKLR1_KO) and a heterozygote for the receptor knockout (HET). Also of importance are the two source facilities: "BZ" and "CJS". It is generally a good idea to include as much metadata as possible, since this data can easily be explored later on.

Commands below assume that 4 CPUs are available for use.


# Stitch paired-end reads together

We first stitch the paired-end reads together using PEAR:

    run_pear.pl -p 4 -o stitched_reads raw_data/*

Four FASTQ files will be generated for each set of paired-end reads: 
(1) assembled reads (used for downstream analyses)
(2) discarded reads (often abnormal reads, e.g. large run of Ns).
(3) unassembled forward reads
(4) unassembled reverse reads

To get an idea of how many reads have been successfully stitched you can compare the sizes of each of the files: 

    cat *.assembled.fastq | wc -l #returned 29539776
    cat *.discarded.fastq | wc -l #return 5224
    cat *.unassembled.forward.fastq | wc -l #returned 405164
    cat *.unassembled.reverse.fastq | wc -l #returned 405164

Note that the return values are the number of lines in each type of FASTQ (when they are concatenated). The number of reads can be calculated by dividing each value by 4. Also, note that the reads in the "assembled" FASTQs are stitched, while the others are not, so the count for these files should be multiplied by 2 for this comparison, resulting in these read counts:

Assembled - 14,769,888 (98.6%)
Discarded - 1,306 (0.0087%)
Unassembled - 202,582 (1.35%)


# Quality metrics of stitched reads

Quality metrics of the 7,384,944 stitched reads can now be determined using FastQC.

This can be done for each sample separately:

    mkdir fastqc_out
    fastqc -t 4 stitched_reads/*.assembled.fastq -o fastqc_out

Or one quality report can be output for all of the stitched reads:

    mkdir fastqc_out_combined
    cat stitched_reads/*.assembled.fastq | fastqc -t 4 stdin -o fastqc_out_combined

    cd fastqc_out_combined
    mv stdin_fastqc.html combined_fastqc.html
    mv stdin_fastqc.zip combined_fastqc.zip

In the output folder(s) an HTML file is created for each FASTQ file (or 1 file in the case of the combined approach above). When you open these HTML files in your browser you can look over a number of quality metrics (a number of the metrics do not give useful results due to the nature of 16S sequencing, read more [here](https://github.com/mlangill/microbiome_helper/wiki/Sequence-QC)). The most informative metric for our purposes is the "Per base sequence quality", which shows the Phred quality score distribution along the reads. Generally these distributions are skewed lower near the 3' read ends. Also, since all reads have the same forward primer sequence there is no variation at the first few positions, as in the below image for all of the stitched FASTQs combined:

![](https://www.dropbox.com/s/v4eaiifsj7jxoyw/combined_stitched_FastQC.jpg?raw=1)

You can see the full FastQC report for all stitched reads combined [here] - **need to add once uploaded on dropbox**.


# Read filtering based on quality and length

Based on the FastQC report above, a quality score cut-off of 30 over 90% of bases and a maximum length of 400 bp are reasonable filtering criteria:
 
    readFilter.pl -q 30 -p 90 -l 400 -thread 4 stitched_reads/*.assembled*fastq

By default this script will output filtered FASTQs in a folder called "filtered_reads" and the percent of reads thrown out after each filtering step is recorded in "readFilter_log.txt".

If you look in this logfile you will note that ~40% of reads were filtered out for each sample. To confirm that the reads were filtered like we wanted we can re-generate the combined FastQC result as before (this would not normally be necessary):

    mkdir fastqc_out_combined_filtered
    cat filtered_reads/*.fastq | fastqc -t 4 stdin -o fastqc_out_combined_filtered

    cd fastqc_out_combined_filtered
    mv stdin_fastqc.html combined_filtered_fastqc.html
    mv stdin_fastqc.zip combined_filtered_fastqc.zip

Here is the corresponding distribution of quality scores after filtering the reads:
![](https://www.dropbox.com/s/s45mmmjx49pnmia/combined_stitched_filtered_FastQC.jpg?raw=1)

As before, if you'd like to see more details (such as how the read length distribution has changed), you can see that [here] - **need to add once uploaded on dropbox**.