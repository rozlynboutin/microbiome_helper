Below is the quick description of our recommended ITS2 pipeline.
  
_Note that this workflow is continually being updated. If you want to use the below commands be sure to keep track of them locally._    
    
_Last updated: 4 July 2017 (see "revisions" above for earlier versions)_    
     
  
*This workflow starts with raw paired-end MiSeq data in demultiplexed FASTQ format assumed to be located within a folder called `raw_data`*

1. As of v2.0.0 of the virtual box we recommend you use a conda environment to run this workflow, which makes sure the correct version of the Python packages required by qiime are being used. You can activate this conda environment with this command:
  
        source activate qiime1

2. (Optional) Run FastQC to allow manual inspection of the quality of sequences

        mkdir fastqc_out
        fastqc -t 4 raw_data/* -o fastqc_out/

3. Stitch paired-end reads together (summary of stitching results are written to "pear_summary_log.txt")

        run_pear.pl -p 4 -o stitched_reads raw_data/* 

4. Filter stitched reads by quality score (at least Q30 over at least 90% of the read), length (at least 200 bp) and ensure forward and reverse primers match 100% each read (summary written to "read_filter_log.txt" by default). If you do not wish to force primer matching, then you must remove the -f/-r/-c options below. 

        read_filter.pl -f GTGAATCATCGAATCTTTGAA -r GCATATCAATAAGCGGAGGA -c both --thread 4 -q 30 -p 90 -l 200 stitched_reads/*.assembled.*
									
5. Convert FASTQ stitched files to FASTA AND remove any sequences that have an 'N' in them.

        run_fastq_to_fasta.pl -p -o fasta_files filtered_reads/*fastq

6. Remove chimeric sequences with VSEARCH (summary written to "chimera_filter_log.txt" by default).

        chimera_filter.pl --thread 4 -type 1 -db /home/shared/rRNA_db/UNITE_uchime_ITS2only_01.01.2016.fasta fasta_files/*	

7. Create a QIIME "map.txt" file with the first column containing the sample names and another column called "FileInput" containing the filenames. This is a tab-delimited file and there must be columns named "BarcodeSequence" and "LinkerPrimerSequence" that are empty. This file can then contain other columns to group samples which will be used when figures are created later.

        create_qiime_map.pl non_chimeras/* > map.txt
		
8. Combine files into single QIIME "seqs.fna" file (~5 minutes).

        add_qiime_labels.py -i non_chimeras/ -m map.txt -c FileInput -o combined_fasta
		
9. Create OTU picking parameter file. We chose to cluster the ITS sequences at the 97% identity which is considered the appropriate level for determining species (see [Blaalid et al., 2013](https://www.ncbi.nlm.nih.gov/pubmed/23350562)).

        echo "pick_otus:threads 4" >> clustering_params.txt
        echo "pick_otus:sortmerna_coverage 0.5" >> clustering_params.txt
        echo "pick_otus:similarity 0.97" >> clustering_params.txt
        echo "assign_taxonomy:id_to_taxonomy_fp /home/shared/rRNA_db/UNITE_sh_refs_qiime_ver7_dynamic_20.11.2016.goodASCII.txt" >> clustering_params.txt 
        echo "assign_taxonomy:reference_seqs_fp /home/shared/rRNA_db/UNITE_sh_refs_qiime_ver7_dynamic_20.11.2016.fasta" >> clustering_params.txt
        echo "assign_taxonomy:assignment_method mothur" >> clustering_params.txt
        
10. Run the entire QIIME open reference picking pipeline with the new sortmerna (for reference picking) and sumaclust (for de novo OTU picking). This does reference picking first, then subsamples failure sequences, _de novo_ OTU picks failures, ref picks against _de novo_ OTUs, and _de novo_ picks again any left over failures. As the ITS2 region is even more variable than 16S/18S, a good reference alignment is not possible for this marker so the alignment and tree-building steps are suppressed. Note: You may want to change the subsampling percentage to a higher amount from the default -s 0.001 to -s 0.01 (e.g 1% of the failures) or -s 0.1 (e.g. 10% of the failures) (~24 hours).

        pick_open_reference_otus.py -i $PWD/combined_fasta/combined_seqs.fna -o $PWD/clustering/ -p $PWD/clustering_params.txt -m sortmerna_sumaclust -s 0.1 -v --min_otu_size 1 -r /home/shared/rRNA_db/UNITE_sh_refs_qiime_ver7_dynamic_20.11.2016.fasta --suppress_align_and_tree

11. Filter OTU table to remove singletons as well as low-confidence OTUs that are likely due to MiSeq bleed-through between runs (reported by Illumina to be 0.1% of reads). 

        remove_low_confidence_otus.py -i $PWD/clustering/otu_table_mc1_w_tax.biom -o $PWD/clustering/otu_table_high_conf.biom

12. Summarize OTU table to determine number of sequences per sample.

        biom summarize-table -i clustering/otu_table_high_conf.biom -o clustering/otu_table_high_conf_summary.txt

13. Normalize OTU table to same sample depth - you will need to change the value of X shown below to match the read count of the sample with the lowest (acceptable) number of reads. Note: Don't like the idea of throwing away all that data? You may want to consider trying different normalization methods such as DESeq2 (see Additional QIIME Analysis in right panel).

        mkdir final_otu_tables
        single_rarefaction.py -i clustering/otu_table_high_conf.biom -o final_otu_tables/otu_table.biom -d X

14. Manually add column(s) to map.txt that contain information to group your samples (e.g. healthy vs disease).

15. Create alpha-diversity rarefaction plot - values min (first point on graph) and max rare depth (last point on graph = your max. above) as well as number of steps (= number of points on graph, not including first min point) should be based on the number of sequences within your OTU table.

        echo "alpha_diversity:metrics observed_otus" >> rarefaction_params.txt
        alpha_rarefaction.py -i final_otu_tables/otu_table.biom -o plots/alpha_rarefaction_plot -p rarefaction_params.txt -m map.txt --min_rare_depth X --max_rare_depth X --num_steps X

16. Make stacked bar charts of either all samples individually or collapsed into your various categories (the -s at the end sorts the samples alphabetically in the charts).

        #For Individual Samples:
        summarize_taxa_through_plots.py -i final_otu_tables/otu_table.biom -o plots/taxa_summary -s
        #Collapsed by Category:
        summarize_taxa_through_plots.py -i final_otu_tables/otu_table.biom -o plots/taxa_summary -m map.txt -c category_type_here -s

17. Convert BIOM OTU table to tab-separated file to be opened/explored in text editors or Excel, etc.

        biom convert -i final_otu_tables/otu_table.biom -o final_otu_tables/otu_table_w_tax.txt --to-tsv --header-key taxonomy

18. Convert BIOM OTU table to STAMP format:

        biom_to_stamp.py -m taxonomy final_otu_tables/otu_table.biom > final_otu_tables/otu_table.spf

19. Fix instances of "unidentified" taxa at intermediate taxonomic levels so STAMP can read in table.

        fix_ITS2_spf.py -i final_otu_tables/otu_table.spf -o final_otu_tables/otu_table_fix.spf 

20. Add sample metadata to BIOM file so that it can be used by other tools like phinch.org and phyloseq.

        biom add-metadata -i final_otu_tables/otu_table.biom -o final_otu_tables/otu_table_with_metadata.biom -m map.txt