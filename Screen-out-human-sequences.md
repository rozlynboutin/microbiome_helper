Our run_human_filter.pl script wraps [Bowtie2](http://bowtie-bio.sourceforge.net/bowtie2/index.shtml) to screen out human sequences. You can use the script like so:
    
    run_human_filter.pl -p 4 -o screened_reads/ stitched_reads/*.assembled*

Note that the hg19 FASTA and Bowtie2 index files need to be in /home/shared/bowtiedb/hg19 and bowtie2 needs to be in your PATH.

Alternatively you could use run_deconseq.pl instead, but it is much slower.