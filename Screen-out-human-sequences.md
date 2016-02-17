Run Bowtie2 to screen out human sequences (Note: you can use run_deconseq.pl instead but it is much slower).
    
    run_human_filter.pl -p 4 -o screened_reads/ stitched_reads/*.assembled*