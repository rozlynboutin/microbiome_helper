Below is the quick and dirty description of our recommended metagenomics pipeline. See the sidebar menu for more detailed descriptions.  
    
_Note that this workflow is continually being updated. If you want to use the below commands be sure to keep track of them locally._   
    
_Last updated: 15 Nov 2016 (see "revisions" above for earlier versions)_   
     
    
1. (Optional) Concatenate multiple lanes of sequencing together (e.g. for NextSeq data). If you do this step remember to change "raw_data" to "concat_data" below.

        concat_lanes.pl raw_data/* -o concat_data -p 4

2. (Optional) Run FastQC to allow manual inspection of the quality of sequences.

        mkdir fastqc_out
        fastqc -t 4 raw_data/* -o fastqc_out/

3. Stich paired end reads together (summary of stitching results are written to "pear_summary_log.txt"). Note: it is important to check the % of reads assembled. It may be better to concatenate the forward and reverse reads together if the assembly % is too low.

        run_pear.pl -p 4 -o stitched_reads raw_data/*

4. Run Bowtie2 to screen out contaminant sequences, here we are screening out reads that map to the human or PhiX genomes (log written to "screened_reads.log" by default). Note: you can use run_deconseq.pl instead but it is much slower.
    
        echo "--local" >> ./bowtie2_config.txt ### add the bowtie2 options you want to use to a config file
        run_contaminant_filter.pl -p 4 -o screened_reads/ stitched_reads/*.assembled* -d /home/shared/bowtiedb/GRCh38_PhiX -c ./bowtie2_config.txt
  
5. Run Trimmomatic to trim off bases under specified quality values and to discard reads under a certain length after trimming (running FastQC on the screened reads is helpful for choosing these parameters). 

        run_trimmomatic.pl -l 5 -t 5 -r 15 -w 4 -m 70 -j /usr/local/prg/Trimmomatic-0.36/trimmomatic-0.36.jar --thread 4 -o trimmomatic_filtered screened_reads/*fastq  
  
6. Run MetaPhlAn2 for taxonomic composition.

        run_metaphlan2.pl -p 4 -o metaphlan_taxonomy.txt trimmomatic_filtered/*fastq
  
7. Convert from MetaPhlAn to STAMP profile file.

        metaphlan_to_stamp.pl metaphlan_taxonomy.txt > metaphlan_taxonomy.spf

8. Run pre-HUMAnN (DIAMOND search).

        run_pre_humann.pl -p 4 -o pre_humann/ trimmomatic_filtered/*fastq

9. Run HUMAnN (link files to HUMAnN "input" directory and then run HUMAnN with scons command). Note that you can run this in parallel with `-j` option (e.g. scons -j 4), but I have found this often causes HUMAnN to unexpectedly error.

        rm /home/shared/humann-0.99/input/*txt 
        rm /home/shared/humann-0.99/output/*

        ln -s $PWD/pre_humann/* /home/shared/humann-0.99/input/
        cd /home/shared/humann-0.99/
        scons

10. Convert HUMAnN output to STAMP format

        cd output/
        humann_to_stamp.pl 04b-hit-keg-mpm-cop-nul-nve-nve.txt > humann_modules.spf
        humann_to_stamp.pl 04b-hit-keg-mpt-cop-nul-nve-nve.txt > humann_pathways.spf
        humann_to_stamp.pl 01b-hit-keg-cat.txt > humann_kos.spf
