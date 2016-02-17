***Human Contamination***
        
* **run_deconseq.pl**: Wraps the deconseq program to filter out human reads from metagenomic data

* **run_human_filter.pl**: Wraps the Bowtie2 program to filter out human reads from metagenomic data (faster than deconseq) 

***PEAR (Paired-end stitching)***

* **run_pear.pl**: Makes running the PEAR program easier on many samples, by automatically identifying the paired-end files to merge together. 

***STAMP (Statistics and Visualization)***

I find STAMP to be a very useful tool for creating figures and doing statistical analyses. Here are scripts to covert tables from different tools into STAMP input profile files.

* **metaphlan_to_stamp.pl**: This is script that converts a merged MetaPhlAn output file that was created using all taxnomic ranks to a STAMP profile file. 

* **biom_to_stamp.py**: STAMP has built-in BIOM conversion, but depending on where the BIOM file comes from there can be slight format problems with STAMP. Specifically, this script handles PICRUSt BIOM output (both KOs and KEGG Pathways) and QIIME 16S OTU tables (with metadata 'taxonomy').

* **humann_to_stamp.pl**: Converts HUMAnN output to STAMP format (currently removes extra rows and renames samples ids so they are the same as the orginal file)

***MetaPhlan (Metagenome taxonomic annotation)***

* **run_metaphlan.pl**: Wraps the MetaPhlAn package and handles running multiple samples at once as well as handling paired end data a bit more cleaner. It also runs each sample in parallel and merges the results into a single output file. Also, easily allows gzipped or non-gzipped files.

***Humann (Metagenome functional annotation)***

* **run_pre_humann.pl**: Does similarity search against kegg database using search tool diamond using multiple threads. This output is then fed into HUMAnN. 

***Other assorted scripts***

* **run_fastq_to_fasta.pl**: Wraps the fastq_to_fasta command from the FASTX Toolkit to allow the use of multiple threads.

* **chimeraFilter.pl**: Wraps UCHIME (implemented in USEARCH) to filter out chimeric reads from a directory of reads in fasta format.  

* **readFilter.pl**: Wraps several read filtering commands together (using FASTX Toolkit and BBMap) to run on a directory of fastq files. 

* **remove_low_confidence_otus.py**: Filters a BIOM table to remove low confidence OTUs that result from MiSeq run-to-run bleed-through (based on 0.1% as reported by Illumina).   