Below is the quick description of our recommended 16S pipeline. See the sidebar menu for more detailed descriptions. 
  
_Note that this workflow is continually being updated. If you want to use the below commands be sure to keep track of them locally._  
  
_Last updated: 4 July 2017 (see "revisions" above for earlier versions)_  
   
*This workflow starts with raw paired-end MiSeq data in demultiplexed FASTQ format assumed to be located within a folder called `raw_data`*

1. As of v2.0.0 of the virtual box we recommend you use a conda environment to run this workflow, which makes sure the correct version of the Python packages required by qiime are being used. You can activate this conda environment with this command:
```
source activate qiime1
```


2. (Optional) Run FastQC to allow manual inspection of the quality of sequences
```
mkdir fastqc_out
fastqc -t 4 raw_data/* -o fastqc_out/
```

3. Stitch paired-end reads together (summary of stitching results are written to "pear_summary_log.txt")

```
run_pear.pl -p 4 -o stitched_reads raw_data/* 
```

4. Filter stitched reads by quality score and length. Summary written to "read_filter_log.txt" by default. Four common options for primer matching are shown below.
  
    * Option A: Read filtering without requiring primer sequences to be at the beginning and end of each read    
  
            read_filter.pl -q 30 -p 90 -l 400 --primer_check none --thread 4 stitched_reads/*.assembled.*  
   
    * Option B: Read filtering with the requirement of forward and reverse primers matching each read for Bacteria-specific V6/V8 amplicons (the primers we use for this region are the default sequences) 
  
            read_filter.pl -c both -q 30 -p 90 -l 400 --thread 4 stitched_reads/*.assembled.*  

    * Option C: Read filtering with primer matching as above, but for Archaea-specific V6/V8 amplicons 
  
            read_filter.pl -f TYAATYGGANTCAACRCC -r TGCYCCTTGYACWCACYG -c both --thread 4 -q 30 -p 90 -l 400 stitched_reads/*.assembled.*  

    * Option D: Read filtering with primer matching as above, but for "universal" V4/V5 amplicons  
  
            read_filter.pl -f GTGYCAGCMGCCGCGGTAA -r AAACTYAAAKRAATTGRCGG -c both --thread 4 -q 30 -p 90 -l 400 stitched_reads/*.assembled.*  
		
5. Convert FASTQ stitched files to FASTA AND remove any sequences that have an 'N' in them.

```
run_fastq_to_fasta.pl -p 4 -o fasta_files filtered_reads/*fastq
```

6. Remove chimeric sequences with VSEARCH (summary written to "chimera_filter_log.txt" by default).

```
chimera_filter.pl --thread 4 -type 1 -db /home/shared/rRNA_db/RDP_trainset16_022016.fa fasta_files/*	
```

7. Create a QIIME "map.txt" file with the first column containing the sample names and another column called "FileInput" containing the filenames. This is a tab-delimited file and there must be columns named "BarcodeSequence" and "LinkerPrimerSequence" that are empty. This file can then contain other columns to group samples which will be used when figures are created later.

```
create_qiime_map.pl non_chimeras/* > map.txt
```
		
8. Combine files into single QIIME "seqs.fna" file (~5 minutes).

```
add_qiime_labels.py -i non_chimeras/ -m map.txt -c FileInput -o combined_fasta
```
		
9. Create OTU picking parameter file.

```
echo "pick_otus:threads 4" >> clustering_params.txt
echo "pick_otus:sortmerna_coverage 0.8" >> clustering_params.txt
```
       
10. Run the entire QIIME open reference picking pipeline with the new sortmerna (for reference picking) and sumaclust (for _de novo_ OTU picking). This does reference picking first, then subsamples failure sequences, _de novo_ OTU picks failures, ref picks against _de novo_ OTUs, and _de novo_ picks again any left over failures. Note: You may want to change the subsampling percentage to a higher amount from the default -s 0.001 to -s 0.01 (e.g 1% of the failures) or -s 0.1 (e.g. 10% of the failures) (~24 hours).

```
pick_open_reference_otus.py -i $PWD/combined_fasta/combined_seqs.fna -o $PWD/clustering/ -p $PWD/clustering_params.txt -m sortmerna_sumaclust -s 0.1 -v --min_otu_size 1 
```

11. Filter OTU table to remove singletons as well as low-confidence OTUs that are likely due to MiSeq bleed-through between runs (reported by Illumina to be 0.1% of reads). 

```
remove_low_confidence_otus.py -i $PWD/clustering/otu_table_mc1_w_tax_no_pynast_failures.biom -o $PWD/clustering/otu_table_high_conf.biom
```

12. Summarize OTU table to determine number of sequences per sample.

```
biom summarize-table -i clustering/otu_table_high_conf.biom -o clustering/otu_table_high_conf_summary.txt
```

13. Normalize OTU table to same sample depth - you will need to change the value of X shown below to match the read count of the sample with the lowest (acceptable) number of reads. Note: Don't like the idea of throwing away all that data? You may want to consider trying different normalization methods such as DESeq2 (see Additional QIIME Analysis in right panel).

```
mkdir final_otu_tables
single_rarefaction.py -i clustering/otu_table_high_conf.biom -o final_otu_tables/otu_table.biom -d X
```

14. Manually add column(s) to map.txt that contain information to group your samples (e.g. healthy vs disease).

15. Create UniFrac beta-diversity plots.

```
beta_diversity_through_plots.py -m map.txt -t clustering/rep_set.tre -i final_otu_tables/otu_table.biom -o plots/bdiv_otu
```

16. Create alpha-diversity rarefaction plot - values min (first point on graph) and max rare depth (last point on graph = your max. above) as well as number of steps (= number of points on graph, not including first min point) should be based on the number of sequences within your OTU table.

```
alpha_rarefaction.py -i final_otu_tables/otu_table.biom -o plots/alpha_rarefaction_plot -t clustering/rep_set.tre -m map.txt --min_rare_depth X --max_rare_depth X --num_steps X
```

17. Make stacked bar charts of either all samples individually or collapsed into your various categories (the -s at the end sorts the samples alphabetically in the charts).

    For Individual Samples:
    ```
    summarize_taxa_through_plots.py -i final_otu_tables/otu_table.biom -o plots/taxa_summary -s
    ```
    Collapsed by Category:
    ```
    summarize_taxa_through_plots.py -i final_otu_tables/otu_table.biom -o plots/taxa_summary -m map.txt -c category_type_here -s
    ```

18. Convert BIOM OTU table to tab-separated file to be opened/explored in text editors or Excel, etc.
```
biom convert -i final_otu_tables/otu_table.biom -o final_otu_tables/otu_table_w_tax.txt --to-tsv --header-key taxonomy
```

19. Convert BIOM OTU table to STAMP. Note: the genus _Clostridium_ is within two families, which can cause downstream problems. To get around this, we add quotations around the genus name so that it is "_Clostridium_" in one of the families.
```        
biom_to_stamp.py -m taxonomy final_otu_tables/otu_table.biom >final_otu_tables/otu_table.spf
sed -i 's/f__Erysipelotrichaceae\tg__Clostridium/f__Erysipelotrichaceae\tg__"Clostridium"/g' final_otu_tables/otu_table.spf  
```

20. Add sample metadata to BIOM file so that it can be used by other tools like phinch.org and phyloseq.
```
biom add-metadata -i final_otu_tables/otu_table.biom -o final_otu_tables/otu_table_with_metadata.biom -m map.txt
```