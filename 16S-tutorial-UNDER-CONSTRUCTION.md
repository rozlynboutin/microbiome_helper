## Introduction

This tutorial will demonstrate how to analyze and interpret Illumina MiSeq 16S sequencing data using the [Microbiome Helper 16S Workflow](https://github.com/mlangill/microbiome_helper/wiki/16S-standard-operating-procedure). It's based on a [previous tutorial we posted](https://github.com/mlangill/microbiome_helper/wiki/16S-tutorial), except this version of the dataset was down-sampled to many fewer reads. This changes some of the results, but makes the tutorial run much faster.

Throughout the tutorial there will be questions to make sure you're understanding. The [answers are here](https://github.com/mlangill/microbiome_helper/wiki/16S-tutorial-answers).

### Requirements
* Basic unix skills (This is a good introductory tutorial: http://korflab.ucdavis.edu/bootcamp.html)
* The exact commands we'll be running assume that you're running this tutorial on our [Ubuntu Desktop virtual box](https://github.com/mlangill/microbiome_helper/wiki/MicrobiomeHelper-Virtual-Box). If you are running it elsewhere just be aware you will need to change the file paths. 
* Download the [tutorial dataset](https://www.dropbox.com/s/r2jqqc7brxg4jhx/16S_chemerin_tutorial.zip?dl=1) (9 MB). I'm assuming the unzipped folder will be in your Desktop.

### Background
This dataset was originally used in a project to determine whether knocking out the protein [chemerin](https://en.wikipedia.org/wiki/Chemerin) affects gut microbial composition. Originally 116 mouse samples acquired from two different facilities were used for this project (**only 24 samples were used in this tutorial dataset, for simplicity**). Metadata associated with each sample is indicated in the mapping file (map.txt). In this mapping file the genotypes of interest can be seen: wildtype (WT and WT_BZ), chemerin knockout (chemerin_KO), chemerin receptor knockout (CMKLR1_KO) and a heterozygote for the receptor knockout (HET). Also of importance are the two source facilities: "BZ" and "CJS". It is generally a good idea to include as much metadata as possible, since this data can easily be explored later on.

## Pre-processing

After downloading and unzipping the dataset above, open Terminal and go to the dataset's directory:

    cd ~/Desktop/16S_chemerin_tutorial

You can see what is in this directory with the _ls_ command:

    ls
    
You should see this:

    vagrant@MicrobiomeHelper:~/Desktop/16S_chemerin_tutorial$ ls
    fastq  map.txt
 
"fastq" is the directory containing all the sequencing files, which we are going to process. "map.txt" contains metadata about the samples. We can look at it with the _less_ command:

    less map.txt

You can see the first column is the sample IDs, the next 2 are blank (note the file is tab-delimited, meaning each column is separated by a tab, not just whitespace) and the 4th column contains fasta filenames (these filenames are based on what _we will_ produce in this pipeline). The rest of the columns are metadata about the samples.

Here is what the first 4 lines should look like:

    #SampleID       BarcodeSequence LinkerPrimerSequence    FileInput       Source  Mouse#  Cage#   genotype        SamplingWeek
    105CHE6WT                       105CHE6WT_S325_L001.assembled_filtered.nonchimera.fasta BZ      BZ25    7       WT      wk6
    106CHE6WT                       106CHE6WT_S336_L001.assembled_filtered.nonchimera.fasta BZ      BZ26    7       WT      wk6
    107CHE6KO                       107CHE6KO_S347_L001.assembled_filtered.nonchimera.fasta BZ      BZ27    7       chemerin_KO     wk6

**Q1)** Based on the _Source_ columnm, 
 
### Stitch paired-end reads together

The raw reads are in [FASTQ format](https://en.wikipedia.org/wiki/FASTQ_format) and (in this case) paired-end. 

We first stitch the paired-end reads together using PEAR (~50 min on 4 CPUs):

    run_pear.pl -p 4 -o stitched_reads raw_data/*

Four FASTQ files will be generated for each set of paired-end reads: 
(1) assembled reads (used for downstream analyses)
(2) discarded reads (often abnormal reads, e.g. large run of Ns).
(3) unassembled forward reads
(4) unassembled reverse reads

To get an idea of how many reads have been successfully stitched you can compare the sizes of each of the files: 

    cat *.assembled.fastq | wc -l #returned 29539776
    cat *.discarded.fastq | wc -l #returned 5224
    cat *.unassembled.forward.fastq | wc -l #returned 405164
    cat *.unassembled.reverse.fastq | wc -l #returned 405164

Note that the return values are the number of lines in each type of FASTQ (when they are concatenated). The number of reads can be calculated by dividing each value by 4. Also, note that the reads in the "assembled" FASTQs are stitched, while the others are not, so the count for these files should be multiplied by 2 for this comparison, resulting in these read counts:

* Assembled - 14,769,888 (98.6%)
* Discarded - 1,306 (0.0087%)
* Unassembled - 202,582 (1.35%)
  
  
### Quality metrics of stitched reads

Quality metrics of the 7,384,944 stitched reads can now be determined using FastQC.

This can be done for each sample separately (~3 min on 4 CPUs):

    mkdir fastqc_out
    fastqc -t 4 stitched_reads/*.assembled.fastq -o fastqc_out

Or one quality report can be output for all of the stitched reads:

    mkdir fastqc_out_combined
    cat stitched_reads/*.assembled.fastq | fastqc -t 4 stdin -o fastqc_out_combined

    cd fastqc_out_combined
    mv stdin_fastqc.html combined_fastqc.html
    mv stdin_fastqc.zip combined_fastqc.zip

In the output folder(s) an HTML file is created for each FASTQ file (or 1 file in the case of the combined approach above). When you open these HTML files in your browser you can look over a number of quality metrics (a number of the metrics do not give useful results due to the nature of 16S sequencing, read more [here](https://github.com/mlangill/microbiome_helper/wiki/Sequence-QC)). The most informative metric for our purposes is the "Per base sequence quality", which shows the Phred quality score distribution along the reads. Generally these distributions are skewed lower near the 3' read ends. Also, since all reads have the same forward primer sequence there is no variation at the first few positions, as in the below image for all of the stitched FASTQs combined:

![](https://www.dropbox.com/s/v4eaiifsj7jxoyw/combined_stitched_FastQC.jpg?raw=1)

You can see the full FastQC report for all stitched reads combined [here](https://www.dropbox.com/s/qm0ush9o8u8s399/combined_fastqc.html).
  
  
### Read filtering based on quality and length

Based on the FastQC report above, a quality score cut-off of 30 over 90% of bases and a maximum length of 400 bp are reasonable filtering criteria (~22 min on 4 CPUs):
 
    readFilter.pl -q 30 -p 90 -l 400 -thread 4 stitched_reads/*.assembled*fastq

By default this script will output filtered FASTQs in a folder called "filtered_reads" and the percent of reads thrown out after each filtering step is recorded in "readFilter_log.txt".

If you look in this logfile you will note that ~40% of reads were filtered out for each sample. To confirm that the reads were filtered like we wanted we can re-generate the combined FastQC result as before (this would not normally be necessary):

    mkdir fastqc_out_combined_filtered
    cat filtered_reads/*.fastq | fastqc -t 4 stdin -o fastqc_out_combined_filtered

    cd fastqc_out_combined_filtered
    mv stdin_fastqc.html combined_filtered_fastqc.html
    mv stdin_fastqc.zip combined_filtered_fastqc.zip

Here is the corresponding distribution of quality scores after filtering the reads:
![](https://www.dropbox.com/s/s45mmmjx49pnmia/combined_stitched_filtered_FastQC.jpg?raw=1)

As before, if you'd like to see more details (such as how the read length distribution has changed), you can see that [here](https://www.dropbox.com/s/d3wbvrdfqpnkpz1/combined_filtered_fastqc.html).

  
### Convert to FASTA and remove chimeric reads

The next steps in the pipeline require the sequences to be in [FASTA format](https://en.wikipedia.org/wiki/FASTA_format), which we will generate using this command (~5 min on 4 CPUs):

    run_fastq_to_fasta.pl -p 4 -o fasta_files filtered_reads/*fastq

Note that this command removes any sequences containing "N" sequences, which is << 1% of the reads after the read filtering steps above.

Now that we have FASTA files we can run the chimera filtering (~3.3 hours on 4 CPUs):

    chimeraFilter.pl -type 1 -thread 4 -db /home/shared/rRNA_db/Bacteria_RDP_trainset15_092015.udb fasta_files/*

You will need to replace "/home/shared/rRNA_db/Bacteria_RDP_trainset15_092015.udb" with your local path to the DB. See a more detailed description of this script [here](https://github.com/mlangill/microbiome_helper/wiki/Remove-chimeric-reads).

This script will remove any reads called as chimeric or called ambiguously and output the remaining reads in the "non_chimeras" folder by default. This step is important for microbiome work since otherwise these reads would be called as novel OTUs (and in fact it is likely that not all chimeric reads will be removed by this step).

By default the logfile "chimeraFilter_log.txt" is generated containing the counts and percentages of reads filtered out for each sample. In this case, ~5-18% of reads are filtered out depending on the sample.


### Run OTU picking pipeline

Now that we have adequately prepared the reads, we can now run OTU picking using QIIME. 
QIIME requires FASTA files to be input in a specific format (specifically, sample names need to be at the beginning of each header line). We have provided the mapping file ("map.txt"), which links filenames to sample names and metadata.

If you open up map.txt (e.g. with vim) you will notice that 2 columns are present without any data: "BarcodeSequence" and "LinkerPrimerSequence". We don't need to use these columns, so we have left them blank. 

Also, you will see that the "FileInput" column contains the names of each FASTA file, which is what we need to specify for the command below.

This command will correctly format the input FASTA files and output a single FASTA:

    add_qiime_labels.py -i non_chimeras/ -m map.txt -c FileInput -o combined_fasta 
  
IF you take a look at combined_fasta/combined_seqs.fna you can see that the first column of header line is a sample name taken from the mapping file.

Now that the input file has been correctly formatted we can run the actual OTU picking program!

Several parameters for this program can be specified into a text file, which will be read in by "pick_open_reference_otus.py":

    echo "pick_otus:threads 4" >> clustering_params.txt
    echo "pick_otus:sortmerna_coverage 0.8" >> clustering_params.txt

We will be using the sortmerna_sumaclust method of OTU picking and subsampling 10% of failed reads for de novo clustering. Lowering the "-s" parameter's value will greatly affect running speed. Also, we are actually retaining singletons (i.e. OTUs identified by 1 read), which we will then remove in the next step. Note that "$PWD" is just a variable that contains your current directory. This command takes ~7.8 hours on 4 CPUs.

    pick_open_reference_otus.py -i $PWD/combined_fasta/combined_seqs.fna -o $PWD/clustering/ -p $PWD/clustering_params.txt -m sortmerna_sumaclust -s 0.1 -v --min_otu_size 1  

We will now remove low confidence OTUs, i.e. those that are called by a low number of reads. It's difficult to choose a hard cut-off for how many reads are needed for an OTU to be confidently called, since of course OTUs are often at low frequency within a community. A reasonable approach is to remove any OTU identified by fewer than 0.1% of the reads (0.1% is the estimated amount of bleed through between runs on the Illumina Miseq):

    remove_low_confidence_otus.py -i $PWD/clustering/otu_table_mc1_w_tax_no_pynast_failures.biom -o $PWD/clustering/otu_table_high_conf.biom

We can compare the summaries of these two BIOM files:

    biom summarize-table -i clustering/otu_table_mc1_w_tax_no_pynast_failures.biom -o clustering/otu_table_mc1_w_tax_no_pynast_failures_summary.txt
    
    biom summarize-table -i clustering/otu_table_high_conf.biom -o clustering/otu_table_high_conf_summary.txt

The first four lines of clustering/otu_table_mc1_w_tax_no_pynast_failures_summary.txt are:
* Num samples: 116
* Num observations: 140817
* Total count: 4205125
* Table density (fraction of non-zero values): 0.032

This means that for the 116 separate samples, 140,817 OTUs were called based on 4,205,125 reads. Only 3.2% of the values in the sample x OTU table are non-zero (meaning that most OTUs are in a small number of samples).

In contrast, the first four lines of  clustering/otu_table_high_conf_summary.txt are:
* Num samples: 116
* Num observations: 4558
* Total count: 3840858
* Table density (fraction of non-zero values): 0.408

The number of OTUs has dropped from 140,817 to 4,558 (kept only 3.2% of the OTUs!). However, the numbers of reads only dropped from 4,204,125 to 3,840,858 (so 91% of the reads were kept!). You can also see that the table density went up by a lot as well, as we would expect.

We now need to subsample the number of reads for each sample to the same depth (rarefaction), which is necessary for several downstream analyses. There is actually quite a lot of debate about whether rarefaction is necessary (since it throws out data!), but it is still the standard method used in microbiome studies. We want to rarify the read depth to the sample with the lowest "reasonable" number of reads. Of course, a "reasonable" read depth is quite subjective and depends on how much variation there is between samples. 

You can look at the read depth per sample in clustering/otu_table_high_conf_summary.txt, here are the first five samples (they are sorted from smallest to largest):

* Counts/sample detail:
* 57CMK8WT: 12452.0
* 31CMK6WT: 12518.0
* 47CMK8KO: 14749.0
* 75CMK8KO: 14760.0
* 105CHE6WT: 15540.0

We will rarify to 12,452 reads, since the lowest depth is not a major outlier:

    single_rarefaction.py -i clustering/otu_table_high_conf.biom -o final_otu_tables/otu_table.biom -d 12452

This QIIME command produced another BIOM table with each sample rarified to 12,452 reads. In this case no OTUs were lost due to this sub-sampling (which you can confirm by producing a summary table), but this step often will result in low-frequency OTUs being lost from the analysis.

### Diversity analyses

We will now create Unifrac beta diversity (both weighted and unweighted) PCA plots:

    beta_diversity_through_plots.py -m map.txt -t clustering/rep_set.tre -i final_otu_tables/otu_table.biom -o plots/bdiv_otu

This QIIME script takes as input the final OTU table as well as file which contains the phylogenetic relatedness between all clustered OTUs. One HTML file will be generated for the weighted and unweighted beta diversity distances:

* plots/bdiv_otu/weighted_unifrac_emperor_pcoa_plot/index.html
* plots/bdiv_otu/unweighted_unifrac_emperor_pcoa_plot/index.html

Open the weighted HTML file in your browser and take a look, you should see this PCA:

![](https://www.dropbox.com/s/dotwkcw37c16jcu/16S_tutorial_weighted_no_meta.jpg?raw=1)

The actual metadata we are most interested in for this dataset is the "genotype" column of the mapping file, which contains the different genotypes I described at the beginning of this tutorial. Go to the "color" tab of the Emperor plot (which is what we were just looking at) and change the selection from "BarcodeSequence" (default) to "genotype". You should see this:

![](https://www.dropbox.com/s/2n2kqv2l15nm2r5/16S_tutorial_weighted_genotype.jpg?raw=1)

Hmm... There are two distinct groups of genotypes, but not the ones we want! It would have made more sense if the WT genotypes all grouped together and the two different chemerin KOs were a separate group. Instead it looks like half of the WTs are split across each group. The fact that all of the WT genotypes called "WT_BZ" gives an important clue of what is going on here - could it be possible that beta diversity is determined much more by which facility the mice came from rather than what genotype they have? To investigate this question we can simply change the selection under the "color" tab from "genotype" to "Source":

![](https://www.dropbox.com/s/n3fzy7g8fkvgjem/16S_tutorial_weighted_source.jpg?raw=1)

Yes, there is a clear qualitative difference between the microbiota of mice from the two source facilities. This can also be seen based on rarefaction plots (alpha diversity plots, which show the OTU richness per sample; takes ~9 min on 4 CPU):

    alpha_rarefaction.py -i final_otu_tables/otu_table.biom -o plots/alpha_rarefaction_plot -t clustering/rep_set.tre -m map.txt --min_rare_depth 1000 --max_rare_depth 12000 --num_steps 12

Note that setting the num_steps higher will give you better resolution, but will take longer. As for the beta diversity plots, you can open the resulting HTML file to view the plots: plots/alpha_rarefaction_plot/alpha_rarefaction_plots/rarefaction_plots.html

Choose "observed_otus" as the metric and "Source" as the category. You should see this plot:

![](https://www.dropbox.com/s/jcbmov0n0zfxc1c/16S_tutorial_alpha_rarefaction_source.jpg?raw=1)

There are clearly more OTUs identified in the guts of mice from the BZ facility than the CJS facility. So what's the next step? Since we know that source facility is such an important factor, we could analyze samples from each facility separately. This will lower our statistical power to detect a signal, but otherwise we cannot easily test for a difference between genotypes. 

To compare the genotypes within the two source facilities separately we fortunately don't need to re-run the OTU-picking. Instead, we can just take different subsets of samples from the final OTU table. First though we need to make two new mapping files with the samples we want to subset:

    head -n 1 map.txt >> map_BZ.txt; awk '{ if ( $3 == "BZ" ) { print $0 } }' map.txt >>map_BZ.txt
    head -n 1 map.txt >> map_CJS.txt; awk '{ if ( $3 == "CJS" ) { print $0 } }' map.txt >>map_CJS.txt

These commands are split into 2 parts (separated by ";"). The first part writes the header line to each new mapping file. The second part is an awk command that prints any line where the 3rd column equals the id of the source facility. Note that awk splits be any whitespace by default, which is why the source facility IDs are in the 3rd column according to awk, even though we know this isn't true when the file is tab-delimited.

The BIOM "subset-table" command requires a text file with 1 sample name per line, which we can generate by these quick bash commands:

    tail -n +2  map_BZ.txt | awk '{print $1}' > samples_BZ.txt
    tail -n +2  map_CJS.txt | awk '{print $1}' > samples_CJS.txt

These commands mean that the first line (the header) should be ignored and then the first column should be printed to a new file. We can now take the two subsets of samples from the BIOM file:

    biom subset-table -j final_otu_tables/otu_table.biom -a sample -s samples_BZ.txt -o final_otu_tables/otu_table_BZ.biom
    biom subset-table -j final_otu_tables/otu_table.biom -a sample -s samples_CJS.txt -o final_otu_tables/otu_table_CJS.biom

 We can now re-create the beta diversity plots for each subset:

    beta_diversity_through_plots.py -m map_BZ.txt -t clustering/rep_set.tre -i final_otu_tables/otu_table_BZ.biom -o plots/bdiv_otu_BZ 
    beta_diversity_through_plots.py -m map_CJS.txt -t clustering/rep_set.tre -i final_otu_tables/otu_table_CJS.biom -o plots/bdiv_otu_CJS 

We can now take a look at the re-generated beta diversity PCAs for each source facility separately.

For the BZ source facility:

![](https://www.dropbox.com/s/yjgc3i9mo0l1pv1/16S_tutorial_weighted_genotype_BZ.jpg?raw=1)

And for the CJS source facility:

![](https://www.dropbox.com/s/f8flg3cwjl1fku1/16S_tutorial_weighted_genotype_CJS.jpg?raw=1)

Just by looking at these PCAs it's clear that if there is any difference it is extremely subtle. To statistically evaluate whether the weighted Unifrac beta diversities differ between genotypes within each source facility, we will two common tests: ANOSIM and ADONIS.

    compare_categories.py -i final_otu_tables/otu_table_BZ.biom --method anosim -i plots/bdiv_otu_BZ/weighted_unifrac_dm.txt -m map_BZ.txt -c genotype -o beta_div_tests
    mv beta_div_tests/anosim_results.txt  beta_div_tests/anosim_results_BZ.txt 

    compare_categories.py -i final_otu_tables/otu_table_CJS.biom --method anosim -i plots/bdiv_otu_CJS/weighted_unifrac_dm.txt -m map_CJS.txt -c genotype -o beta_div_tests
    mv beta_div_tests/anosim_results.txt  beta_div_tests/anosim_results_CJS.txt 

    compare_categories.py -i final_otu_tables/otu_table_BZ.biom --method adonis -i plots/bdiv_otu_BZ/weighted_unifrac_dm.txt -m map_BZ.txt -c genotype -o beta_div_tests
    mv beta_div_tests/adonis_results.txt  beta_div_tests/adonis_results_BZ.txt 

    compare_categories.py -i final_otu_tables/otu_table_CJS.biom --method adonis -i plots/bdiv_otu_CJS/weighted_unifrac_dm.txt -m map_CJS.txt -c genotype -o beta_div_tests
    mv beta_div_tests/adonis_results.txt  beta_div_tests/adonis_results_CJS.txt 

You can take a look at the output files to see significance values and test statistics. The P-values for all four of these tests is > 0.05 so there is no significant difference in the Unifrac beta diversities of different genotypes within each source facility.