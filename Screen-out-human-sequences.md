Run [Bowtie2](http://bowtie-bio.sourceforge.net/bowtie2/index.shtml) to screen out human sequences 
    
    run_human_filter.pl -p 4 -o screened_reads/ stitched_reads/*.assembled*


(Note: you can use run_deconseq.pl instead but it is much slower).