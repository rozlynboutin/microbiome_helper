Below is the quick and dirty description of our recommended metagenomics pipeline. See the sidebar menu for more detailed descriptions.

1. (Optional) Run FastQC to allow manual inspection of the quality of sequences.

        mkdir fastqc_out
        fastqc -t 4 raw_data/* -o fastqc_out/

2. Stich paired end reads together (summary of stitching results are written to "pear_summary_log.txt").

        run_pear.pl -p 4 -o stitched_reads raw_data/*

3. Run Bowtie2 to screen out human sequences (Note: you can use run_deconseq.pl instead but it is much slower).
    
        run_human_filter.pl -p 4 -o screened_reads/ stitched_reads/*.assembled*

4. Run MetaPhlAn2 for taxonomic composition.

        run_metaphlan2.pl -p 4 -o metaphlan_taxonomy.txt screened_reads/*

5. Convert from MetaPhlAn to STAMP profile file.

        metaphlan_to_stamp.pl metaphlan_taxonomy.txt > metaphlan_taxonomy.spf

6. Run pre-HUMAnN (DIAMOND search).

        run_pre_humann.pl -p 4 -o pre_humann/ screened_reads/*

7. Run HUMAnN (link files to HUMAnN "input" directory and then run HUMAnN with scons command). Note that you can run this in parallel with `-j` option (e.g. scons -j 4), but I have found this often causes HUMAnN to unexpectedly error.

        ln -s $PWD/pre_humann/* ~/programs/humann-0.99/input/
        cd ~/programs/humann-0.99/
        scons

8. Convert HUMAnN output to STAMP format

        humann_to_stamp.pl 04b-hit-keg-mpm-cop-nul-nve-nve.txt > hummann_modules.spf
        humann_to_stamp.pl 04b-hit-keg-mpt-cop-nul-nve-nve.txt > hummann_pathways.spf
        humann_to_stamp.pl 01b-hit-keg-cat.txt > hummann_kos.spf
