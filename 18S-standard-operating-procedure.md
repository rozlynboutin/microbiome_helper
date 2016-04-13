Below is the quick and dirty description of our recommended 18S pipeline. See the sidebar menu for more [detail](https://github.com/mlangill/microbiome_helper/wiki/Changes-to-16S-workflow-for-18S-data).

*Note that this workflow starts with raw paired-end MiSeq data in demultiplexed fastq format assumed to be located within a folder called `raw_data`*

1. (Optional) Run FastQC to allow manual inspection of the quality of sequences

        mkdir fastqc_out
        fastqc -t 4 raw_data/* -o fastqc_out/

2. Stich paired end reads together (summary of stitching results are written to "pear_summary_log.txt")

        run_pear.pl -p 4 -o stitched_reads raw_data/* 

3. Filter stitched reads by quality score, length and ensure forward and reverse primers match each read (summary written to "readFilter_log.txt" by default).

        readFilter.pl -f CYGCGGTAATTCCAGCTC -r CRAAGAYGATYAGATACCRT -thread 4 -q 30 -p 90 -l 200 stitched_reads/*.assembled.*
									
4. Convert FASTQ stitched files to FASTA AND remove any sequences that have an 'N' in them.

        run_fastq_to_fasta.pl -p -o fasta_files filtered_reads/*fastq

5. Remove chimeric sequences with UCHIME (summary written to "chimeraFilter_log.txt" by default).

        chimeraFilter.pl -thread 4 -type 1 -db /home/shared/rRNA_db/Eukaryota_SILVA_123_SSURef_Nr99_tax_silva_U-replaced.udb fasta_files/*	

6. Create a QIIME "map.txt" file with the first column containing the sample names and another column called "FileInput" containing the filenames. This is a tab-delimited file and there must be columns named "BarcodeSequence" and "LinkerPrimerSequence" that are empy. This file can then contain other columns to group samples which will be used when figures are created later.

        create_qiime_map.pl non_chimeras/* > map.txt
		
7. Combine files into single QIIME "seqs.fna" file (~5 minutes).

        add_qiime_labels.py -i non_chimeras/ -m map.txt -c FileInput -o combined_fasta
		
8. Create OTU picking parameter file.

        echo "pick_otus:threads 4" >> clustering_params.txt
        echo "pick_otus:sortmerna_coverage 0.8" >> clustering_params.txt
        echo "pick_otus:similarity 0.99" >> clustering_params.txt
        echo "align_seqs:template_fp /home/shared/rRNA_db/Silva_119_rep_set90_aligned_18S.fna" >> clustering_params.txt 
        echo "assign_taxonomy:id_to_taxonomy_fp /home/shared/rRNA_db/Silva_119_rep_set99_18S_taxonomy_7_levels.txt" >> clustering_params.txt
        echo "assign_taxonomy:reference_seqs_fp /home/shared/rRNA_db/Silva_119_rep_set99_18S.fna" >> clustering_params.txt
        
9. Run the entire qiime open reference picking pipeline with the new sortmerna (for reference picking) and sumaclust (for de novo OTU picking). This does reference picking first, then subsamples failure sequences, de-novo OTU picks failures, ref picks against de novo OTUs, and de-novo picks again any left over failures. Note: You may want to change the subsampling percentage to a higher amount from the default -s 0.001 to -s 0.01 (e.g 1% of the failures) or -s 0.1 (e.g. 10% of the failures) (~24 hours).

        pick_open_reference_otus.py -i $PWD/combined_fasta/combined_seqs.fna -o $PWD/clustering/ -p $PWD/clustering_params.txt -m sortmerna_sumaclust -s 0.1 -v --min_otu_size 1 -r /home/shared/rRNA_db/Silva_119_rep_set99_18S.fna

10. Filter OTU table to remove singletons as well as low-confidence OTUs that are likely due to MiSeq bleed-through between runs (reported by Illumina to be 0.1% of reads). 

        remove_low_confidence_otus.py -i $PWD/clustering/otu_table_mc1_w_tax_no_pynast_failures.biom -o $PWD/clustering/otu_table_high_conf.biom

11. Summarize OTU table to determine number of sequences per sample.

        biom summarize-table -i clustering/otu_table_high_conf.biom -o clustering/otu_table_high_conf_summary.txt

12. Normalize OTU table to same sample depth - you will need to change the value of X shown below to match the read count of the sample with the lowest (acceptable) number of reads. Note: Don't like the idea of throwing away all that data? You may want to consider trying different normalization methods such as DESeq2 (see below).

        mkdir final_otu_tables
        single_rarefaction.py -i clustering/otu_table_high_conf.biom -o final_otu_tables/otu_table.biom -d X

13. Manually add column(s) to map.txt that contain information to group your samples (e.g. healthy vs disease).

14. Create UniFrac beta diversity plots.

        beta_diversity_through_plots.py -m map.txt -t clustering/rep_set.tre -i final_otu_tables/otu_table.biom -o plots/bdiv_otu

15. Create alpha diversity rarefaction plot (values min and max rare depth as well as number of steps should be based on the number of sequences within your OTU table).

        alpha_rarefaction.py -i final_otu_tables/otu_table.biom -o plots/alpha_rarefaction_plot -t clustering/rep_set.tre -m map.txt --min_rare_depth 1000 --max_rare_depth 35000 --num_steps 35

16. Convert BIOM OTU table to tab-separated file to be opened/explored in text editors or Excel, etc.

        biom convert -i final_otu_tables/otu_table.biom -o final_otu_tables/otu_table_w_tax.txt --to-tsv --header-key taxonomy

17. Convert BIOM OTU table to STAMP. Note that the current 18S reference database we are using has some naming conflicts, which we are temporarily getting around with the below commands:

        TmpFixBIOM.pl final_otu_tables/otu_table_w_tax.txt  > final_otu_tables/otu_table_w_tax_cleaned.txt

        biom convert -i  final_otu_tables/otu_table_w_tax_cleaned.txt -o final_otu_tables/otu_table_cleaned.biom --to-hdf5 --process-obs-metadata taxonomy --table-type="OTU table"

        biom_to_stamp.py -m taxonomy final_otu_tables/otu_table_cleaned.biom >final_otu_tables/otu_table_w_tax_cleaned.spf

        sed -i 's/D_4__Chlorarachniophyta\tD_5__NOR26/D_4__Chlorarachniophyta\tD_5__NOR26_Chlorarachniophyta/g’  final_otu_tables/otu_table_w_tax_cleaned.spf

18. Add sample metadata to BIOM file so that it can be used by other tools like phinch.org and phyloseq.

        biom add-metadata -i final_otu_tables/otu_table_cleaned.biom -o final_otu_tables/otu_table_cleaned_with_metadata.biom -m map.txt