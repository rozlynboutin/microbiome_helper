We use [FastQC](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/) to check the quality of the sequenced reads. This is especially important after stitching the paired-end reads to help choose quality cut-offs.

To run FastQC on all of your FASTQs separately you just need to run:

    mkdir fastqc_out
    fastqc stitched_reads/*.assembled.fastq -o fastqc_out

Alternatively, if you want to look at the QC metrics for all of your FASTQs combined you can run:

    mkdir fastqc_out_combined
    cat stitched_reads/*.assembled.fastq | fastqc stdin -o fastqc_out_combined

See example output [here](https://www.dropbox.com/s/97n67yvah7x9ncb/combined_fastqc.html) for 16S reads. 

Note that for 16S data all of the reads should begin with same forward primer sequence, which explains why the "Per base sequence content" plot has peaks of 100% base content in the first few positions.

Also, for 16S data, note that a number of metrics ("Sequence Duplication Levels" , "Overrepresented sequences" and "Kmer Content") should not be used to evaluate data quality since of course we are looking at sequencing data for only a single gene, so an excess of highly similar sequences are expected.