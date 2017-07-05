* [Introduction](#introduction)
    * [Requirements](#requirements)
    * [Background](#background)
    * [Initial Set-up](#initial-set-up)
* [Pre-processing](#pre-processing)
    * [Stitching paired-end reads](#stitching-paired-end-reads)
    * [Quality metrics of stitched reads](#quality-metrics-of-stitched-reads)
    * [Filtering reads by quality and length](#filtering-reads-by-quality-and-length)
    * [Conversion to FASTA and removal of chimeric reads](#conversion-to-fasta-and-removal-of-chimeric-reads)
* [Run open-reference OTU picking pipeline](#run-open-reference-otu-picking-pipeline)
    * [Remove low confidence OTUs](#remove-low-confidence-otus)
    * [Rarify reads](#rarify-reads)
* [Diversity analyses](#diversity-analyses)
    * [UniFrac beta diversity analysis](#unifrac-beta-diversity-analysis)
    * [Alpha diversity analysis](#alpha-diversity-analysis)
    * [Testing for statistical differences](#testing-for-statistical-differences)
* [Using STAMP to test for particular differences](#using-stamp-to-test-for-particular-differences)

## Introduction

This tutorial outlines how to process 16S rRNA sequencing data with the [Microbiome Helper 16S Workflow](https://github.com/mlangill/microbiome_helper/wiki/16S-Bacteria-and-Archaea-Standard-Operating-Procedure).

The tutorial is split into 3 main parts:  
* pre-processing (stitching paired-end reads, measuring their quality, filtering those which failed to meet standards of quality and length, and removing chimeric reads)
* OTU picking (using QIIME to assign taxonomies, removing OTUs called by very few reads, and rarifying to a reasonable read depth)
* Some downstream diversity analyses. 

This tutorial is based on a previous tutorial we posted, except this version of the dataset was down-sampled to many fewer reads (you can download the original raw reads [here](https://www.dropbox.com/s/4fqgi6t3so69224/Sinal_Langille_raw_data.tar.gz?dl=1)).

Throughout the tutorial there will be questions to help you learn. The [answers are here](https://github.com/mlangill/microbiome_helper/wiki/16S-tutorial-answers).

Also, be aware that for real datasets the run-times (and memory requirements) for these commands will greatly increase.

__Author:__ Gavin Douglas   

__Contributions by:__ Molly Hayes, Morgan Langille

__First created:__ Fall 2015  

__Last edited:__ June 2017  

### Requirements
* Basic unix skills (This is a good introductory tutorial: http://korflab.ucdavis.edu/bootcamp.html)
* The exact commands we'll be running assume that you're running this tutorial on our [Ubuntu Desktop virtual box](https://github.com/mlangill/microbiome_helper/wiki/MicrobiomeHelper-Virtual-Box). If you are running it elsewhere just be aware you will need to change the file paths. 
* Download the [tutorial dataset](https://www.dropbox.com/s/r2jqqc7brxg4jhx/16S_chemerin_tutorial.zip?dl=1) (9 MB). I'm assuming the unzipped folder will be in your Desktop.
* We will be following [our 16S pipeline](https://github.com/mlangill/microbiome_helper/wiki/16S-Bacteria-and-Archaea-Standard-Operating-Procedure), which uses several wrapper scripts to help automate running many files.

### Background
16S analysis is a method of microbiome analysis (compared to [shotgun metagenomics](https://github.com/mlangill/microbiome_helper/wiki/Metagenomics-Tutorial-%28Downsampled%29)) that targets the 16S ribosomal RNA gene, as this gene is present in all prokaryotes. It features regions that are conserved among these organisms, as well as variable regions that allow distinction among organisms. These characteristics make this gene useful for analyzing microbial communities at reduced cost compared to metagenomic techniques. A similar workflow can be applied to eukaryotic micro-organisms using the 18S rRNA gene.

This dataset was originally used in a project to determine whether knocking out the protein [chemerin](https://en.wikipedia.org/wiki/Chemerin) affects gut microbial composition. Originally 116 mouse samples acquired from two different facilities were used for this project (**only 24 samples were used in this tutorial dataset, for simplicity**). Metadata associated with each sample is indicated in the mapping file (map.txt). In this mapping file the genotypes of interest can be seen: wildtype (WT), chemerin knockout (chemerin_KO), chemerin receptor knockout (CMKLR1_KO) and a heterozygote for the receptor knockout (HET). Also of importance are the two source facilities: "BZ" and "CJS". It is generally a good idea to include as much metadata as possible, since this data can easily be explored later on.

## Bug-fix

If you get an error referring to "ImportError: No module named externals" when running add_qiime_labels.py then run the below command. 

    wget http://graal.ift.ulaval.ca/adrouin/patch_matplotlib.sh && sh ./patch_matplotlib.sh

## Initial Set-up
Open a terminal window (by clicking on the little black box icon).

Now we need to download the data using 'wget':

    wget https://www.dropbox.com/s/r2jqqc7brxg4jhx/16S_chemerin_tutorial.zip?dl=1 -O 16S_chemerin_tutorial.zip

Now decompress the data using "unzip" command and change into that directory:

    unzip 16S_chemerin_tutorial.zip
    cd 16S_chemerin_tutorial 

## Browse data

You can see what is in this directory with the _ls_ command:

    ls  
    
You should see two names: "fastq" is the directory containing all the sequencing files, which we are going to process. "map.txt" contains metadata about the samples. We can look at it with the _less_ command (hit "q" to exit):

    less map.txt

You can see the first column is the sample IDs, the next 2 are blank (note the file is tab-delimited, meaning each column is separated by a tab, not just whitespace) and the 4th column contains FASTA filenames (these filenames are based on what _we will_ produce in this pipeline). The rest of the columns are metadata about the samples.

Here is what the first 4 lines should look like:

    #SampleID       BarcodeSequence LinkerPrimerSequence    FileInput       Source  Mouse#  Cage#   genotype        SamplingWeek
    105CHE6WT                       105CHE6WT_S325_L001.assembled_filtered.nonchimera.fasta BZ      BZ25    7       WT      wk6
    106CHE6WT                       106CHE6WT_S336_L001.assembled_filtered.nonchimera.fasta BZ      BZ26    7       WT      wk6
    107CHE6KO                       107CHE6KO_S347_L001.assembled_filtered.nonchimera.fasta BZ      BZ27    7       chemerin_KO     wk6

**Q1)** Based on the _Source_ column, how many samples are from each of the "BZ" and "CJS" source facilities?

Now list the filenames in the fastq folder. You can get the number of lines in a file with this command (where "FILENAME" should a file):

    wc -l FILENAME 

**Q2)** How many reads are in each file? Note that the raw reads are in [FASTQ format](https://en.wikipedia.org/wiki/FASTQ_format). 

You can get the number of FASTQ files in the directory with this command:

    ls *fastq | wc -l

**Q3)** Why isn't the number of FASTQ files equal to the number of samples?

## Stitching paired-end reads 

To start processing the data, we first need to stitch the paired-end reads together using PEAR (~1 min on 1 CPU):

    run_pear.pl -p 1 -o stitched_reads fastq/*fastq

("-p 1" indicates this job should be run on 1 CPU and "-o stitched_reads" indicates that the output folder).

Four FASTQ files will be generated for each set of paired-end reads:   
(1) assembled reads (used for downstream analyses)  
(2) discarded reads (often abnormal reads, e.g. large run of Ns).   
(3) unassembled forward reads   
(4) unassembled reverse reads   
  
The default log file "pear_summary_log.txt" contains the percent of reads either assembled, discarded or unassembled. 

**Q4)** What percent of reads were successfully stitched for sample 40CMK6WT?  
  
### Quality metrics of stitched reads

Quality metrics of the stitched reads can now be determined using FastQC. In a desktop environment you run FastQC through its graphical user interface (GUI), which is on the sidebar in our virtual box or you can open by typing "fastqc" at the command-line. I prefer using the command-line, which I describe below, but you can get the same plots with the GUI.

FastQC can be run for each sample separately (~3 min on 1 CPU):

    mkdir fastqc_out
    fastqc -t 1 stitched_reads/*.assembled.fastq -o fastqc_out

Or one quality report can be output for all of the stitched reads (< 1 min on 1 CPU):

    mkdir fastqc_out_combined
    cat stitched_reads/*.assembled.fastq | fastqc -t 1 stdin -o fastqc_out_combined

    cd fastqc_out_combined

For clarity you should rename the files when using this approach: 

    mv stdin_fastqc.html combined_fastqc.html  
    mv stdin_fastqc.zip combined_fastqc.zip  

In the output folder(s) an HTML file is created for each FASTQ file (or 1 file in the case of the combined approach above). When you open these HTML files in your browser you can look over a number of quality metrics (a number of the metrics do not give useful results due to the nature of 16S sequencing, read more [here](https://github.com/mlangill/microbiome_helper/wiki/Sequence-QC)). The most informative metric for our purposes is the "Per base sequence quality", which shows the Phred quality score -- a measure based on the probabilities of base-calling errors by the sequencer -- distribution along the reads. Generally these distributions are skewed lower near the 3' read ends. Also, since all reads have the same forward primer sequence there is no variation at the first few positions, as in the below image for all of the stitched FASTQs combined:

![](https://www.dropbox.com/s/0xdtcs05q1sfev1/combined_fastqc.png?raw=1)

You can see the full FastQC report for all stitched reads combined [here](https://www.dropbox.com/s/rs3jmeao3s9vaip/combined_fastqc.html).
  
### Filtering reads by quality and length

Based on the FastQC report above, a quality score cut-off of 30 over 90% of bases and a maximum length of 400 bp are reasonable filtering criteria (~2 min on 1 CPU):

    cd ~/Desktop/16S_chemerin_tutorial
 
    read_filter.pl -q 30 -p 90 -l 400 -thread 1 -c both stitched_reads/*.assembled*fastq

The "-c both" option above checks for the default forward and reverse degenerate primer sequences to match exactly in each sequences (You can use whatever primer sequences you want. However, the primer sequences in this case are set by default in the script, which you can look at with "read_filter.pl -h"). If you don't set the "-c" option to either "both" or "forward" then there wont be any primer matching.

By default this script will output filtered FASTQs in a folder called "filtered_reads" and the percent of reads thrown out after each filtering step is recorded in "read_filter_log.txt". This script is just a convenient way to run two popular tools for read filtering: [FASTX-toolkit](http://hannonlab.cshl.edu/fastx_toolkit/) and [BBMAP](https://sourceforge.net/projects/bbmap/).

If you look in this logfile you will note that ~40% of reads were filtered out for each sample. You can also see the counts and percent of reads dropped at each step. 

**Q5)** How many of sample 36CMK6WT's reads were filtered out for not containing a match to the forward primer (which is the default setting in this case).

To confirm that the reads were filtered like we wanted we can re-generate the combined FastQC result as before (this would not normally be necessary):

    mkdir fastqc_out_combined_filtered
    cat filtered_reads/*.fastq | fastqc -t 1 stdin -o fastqc_out_combined_filtered

    cd fastqc_out_combined_filtered
    mv stdin_fastqc.html combined_filtered_fastqc.html
    mv stdin_fastqc.zip combined_filtered_fastqc.zip

Here is the corresponding distribution of quality scores after filtering the reads:
![](https://www.dropbox.com/s/aa6isdfhtg5lahd/combined_fastqc_filtered.png?raw=1)

As before, if you'd like to see more details (such as how the read length distribution has changed), you can see that [here](https://www.dropbox.com/s/bg6mzz6gawlpa7q/combined_filtered_fastqc.html).

  
### Conversion to FASTA and removal of chimeric reads

The next steps in the pipeline require the sequences to be in [FASTA format](https://en.wikipedia.org/wiki/FASTA_format), which we will generate using this command (< 1 min on 1 CPU):

    cd ~/Desktop/16S_chemerin_tutorial  
  
    run_fastq_to_fasta.pl -p 1 -o fasta_files filtered_reads/*fastq

Note that this command removes any sequences containing "N" (a fully ambiguous base read), which is << 1% of the reads after the read filtering steps above.

During PCA amplification 16S rRNA sequences from different organisms can sometimes combine to form hybrid molecules called chimeric sequences. It's important to remove these so they aren't incorrectly called as novel OTUs. Unfortunately, not all chimeric reads will be removed during this step, which is something to keep in mind during the next steps. 

You can run chimera checking with [VSEARCH](https://github.com/torognes/vsearch) with this command (~3 min on 1 CPU):

    chimera_filter.pl -type 1 -thread 1 -db /home/shared/rRNA_db/Bacteria_RDP_trainset15_092015.fa fasta_files/*fasta

See a more detailed description of this script [here](https://github.com/mlangill/microbiome_helper/wiki/Remove-chimeric-reads).

This script will remove any reads called either ambiguously or as chimeric, and output the remaining reads in the "non_chimeras" folder by default.

By default the logfile "chimeraFilter_log.txt" is generated containing the counts and percentages of reads filtered out for each sample. 

**Q6)** What is the mean percent of reads retained after this step, based on the output in the log file ("nonChimeraCallsPercent" column)?

**Q7)** What percent of stitched reads was retained for sample 75CMK8KO after **all** the filtering steps (HINT: you'll need to compare the original number of reads to the number of reads output by chimera_filter.pl)?

## Run open-reference OTU picking pipeline

Now that we have adequately prepared the reads, we can now run OTU picking using QIIME. An Operational Taxonomic Unit (OTU) defines a taxonomic group based on sequence similarity among sampled organisms. QIIME software clusters sequence reads from microbial communities in order to classify its constituent micro-organisms into OTUs. QIIME requires FASTA files to be input in a specific format (specifically, sample names need to be at the beginning of each header line). We have provided the mapping file ("map.txt"), which links filenames to sample names and metadata.

If you open up map.txt (e.g. with vim) you will notice that 2 columns are present without any data: "BarcodeSequence" and "LinkerPrimerSequence". We don't need to use these columns, so we have left them blank. 

Also, you will see that the "FileInput" column contains the names of each FASTA file, which is what we need to specify for the command below.

This command will correctly format the input FASTA files and output a single FASTA:

    add_qiime_labels.py -i non_chimeras/ -m map.txt -c FileInput -o combined_fasta 
  
If you take a look at combined_fasta/combined_seqs.fna you can see that the first column of header line is a sample name taken from the mapping file.

Now that the input file has been correctly formatted we can run the actual OTU picking program!

Several parameters for this program can be specified into a text file, which will be read in by "pick_open_reference_otus.py":

    echo "pick_otus:threads 1" >> clustering_params.txt
    echo "pick_otus:sortmerna_coverage 0.8" >> clustering_params.txt
    echo "pick_otus:sortmerna_db /home/shared/pick_otu_indexdb_rna/97_otus" >> clustering_params.txt

We will be using the sortmerna_sumaclust method of [open-reference](http://qiime.org/scripts/pick_open_reference_otus.html) OTU picking. In open-reference OTU picking, reads are first clustered against a reference database; then, a certain percent (10% in the below command) of those reads that failed to be classified are sub-sampled to create a new reference database and the remaining unclassified reads are clustered against this new database. This de novo clustering step is repeated again by default using the below command (can be turned off to save time with the "–suppress_step4" option).   
   
Also, we are actually retaining singletons (i.e. OTUs identified by 1 read), which we will then remove in the next step. Note that "$PWD" is just a variable that contains your current directory. This command takes ~7 min with 1 CPU (note you may run into problems if the virtual box isn't using at least 2GB of RAM). Lowering the "-s" parameter's value will greatly affect running speed.

    pick_open_reference_otus.py -i $PWD/combined_fasta/combined_seqs.fna -o $PWD/clustering/ -p $PWD/clustering_params.txt -m sortmerna_sumaclust -s 0.1 -v --min_otu_size 1  

### Remove low confidence OTUs

We will now remove low confidence OTUs, i.e. those that are called by a low number of reads. It's difficult to choose a hard cut-off for how many reads are needed for an OTU to be confidently called, since of course OTUs are often at low frequency within a community. A reasonable approach is to remove any OTU identified by fewer than 0.1% of the reads, given that 0.1% is the estimated amount of sample bleed-through between runs on the Illumina Miseq:

    remove_low_confidence_otus.py -i $PWD/clustering/otu_table_mc1_w_tax_no_pynast_failures.biom -o $PWD/clustering/otu_table_high_conf.biom

Since we are just doing a test run with few sequences, the threshold is 1 OTU regardless. However, this is an important step with real datasets.

We can compare the summaries of these two BIOM files:

    biom summarize-table -i clustering/otu_table_mc1_w_tax_no_pynast_failures.biom -o clustering/otu_table_mc1_w_tax_no_pynast_failures_summary.txt
    
    biom summarize-table -i clustering/otu_table_high_conf.biom -o clustering/otu_table_high_conf_summary.txt

The first four lines of clustering/otu_table_mc1_w_tax_no_pynast_failures_summary.txt are:
   
    Num samples: 24
    Num observations: 2420
    Total count: 12014
    Table density (fraction of non-zero values): 0.097

This means that for the 24 separate samples, 2420 OTUs were called based on 12014 reads. Only 9.7% of the values in the sample x OTU table are non-zero, meaning that most OTUs are in a small number of samples.

In contrast, the first four lines of  clustering/otu_table_high_conf_summary.txt are:

    Num samples: 24
    Num observations: 884
    Total count: 10478
    Table density (fraction of non-zero values): 0.193

After removing low-confidence OTUs, only 36.5% were retained: the number of OTUs dropped from 2420 to 884. This effect is generally even more drastic for bigger datasets. However, the numbers of reads only dropped from 12014 to 10478 (so 87% of the reads were retained). You can also see that the table density increased, as we would expect.

### Rarify reads

We now need to subsample the number of reads for each sample to the same depth, which is necessary for several downstream analyses. This is called rarefaction, a technique that provides an indication of species richness for a given number of samples. There is actually quite a lot of debate about whether rarefaction is necessary (since it throws out data!), but it is still the standard method used in microbiome studies. We want to rarify the read depth to the sample with the lowest "reasonable" number of reads. Of course, a "reasonable" read depth is quite subjective and depends on how much variation there is between samples. 

You can look at the read depth per sample in clustering/otu_table_high_conf_summary.txt, here are the first five samples (they are sorted from smallest to largest):

    Counts/sample detail:
    106CHE6WT: 375.0
    111CHE6KO: 398.0
    39CMK6KO: 410.0
    113CHE6WT: 412.0
    108CHE6KO: 413.0

**Q8)** What is the read depth for sample "75CMK8KO"?

**Note that this is a test dataset and you normally would rarify to a larger read count** (typically in the 1000s).

We will rarify to 375 reads, since the lowest depth is not a major outlier in this dataset:

    mkdir final_otu_tables
    single_rarefaction.py -i clustering/otu_table_high_conf.biom -o final_otu_tables/otu_table.biom -d 375

This QIIME command produced another BIOM table with each sample rarified to 375 reads. In this case, no OTUs were lost due to this sub-sampling (which you can confirm by producing a summary table), but this step often will result in a loss of low-frequency OTUs from the analysis.

## Diversity analyses

Diversity in microbial samples can be expressed in a number of ways. Most commonly people refer to "alpha" (the diversity within a group) and "beta" (the diversity between groups) diversity. There are many different ways to compute both of these diversities.     

### UniFrac beta diversity analysis  

[UniFrac](https://en.wikipedia.org/wiki/UniFrac) is a particular beta-diversity measure that analyzes dissimilarity between samples, sites, or communities. We will now create UniFrac beta diversity (both weighted and unweighted) principal coordinates analysis (PCoA) plots. PCoA plots are related to principal components analysis (PCA) plots, but are based on any dissimilarity matrix rather than just a covariance/correlation matrix (< 1 min on 1 CPU):

    beta_diversity_through_plots.py -m map.txt -t clustering/rep_set.tre -i final_otu_tables/otu_table.biom -o plots/bdiv_otu

This QIIME script takes as input the final OTU table, as well as file which contains the phylogenetic relatedness between all clustered OTUs. One HTML file will be generated for the weighted and unweighted beta diversity distances:

* plots/bdiv_otu/weighted_unifrac_emperor_pcoa_plot/index.html
* plots/bdiv_otu/unweighted_unifrac_emperor_pcoa_plot/index.html

Open the weighted HTML file in your browser and take a look, you should see a PCoA very similar to this:

![](https://www.dropbox.com/s/ryamzk9mp7kbbkz/raw_beta.png?raw=1)

The actual metadata we are most interested in for this dataset is the "genotype" column of the mapping file, which contains the different genotypes I described at the beginning of this tutorial. Go to the "Colors" tab of the Emperor plot (which is what we were just looking at) and change the selection from "BarcodeSequence" (default) to "genotype". You should see a similar plot to this:

![](https://www.dropbox.com/s/uq4vqse74p1fe4k/raw_geno.png?raw=1)

The WT genotype is spread out across both knockout genotypes, which is not what we would have predicted.  
   
You'll see what's really driving the differences in beta diversity when you change the selection under the "Colors" tab from "genotype" to "Source":

![](https://www.dropbox.com/s/1dr42gaezzjf0ay/raw_sources.png?raw=1)

### Alpha diversity analysis

There is a clear qualitative difference between the microbiota of mice from the two source facilities based on the above plots. It's also possible that there might also be differences between the rarefaction curves of samples from different source facilities. Rarefaction plots show alpha diversity, which in this case is taken as a measure of the OTU richness per sample (takes < 1 min on 1 CPU):

    alpha_rarefaction.py -i final_otu_tables/otu_table.biom -o plots/alpha_rarefaction_plot -t clustering/rep_set.tre -m map.txt --min_rare_depth 40 --max_rare_depth 375 --num_steps 10

Note that setting the num_steps higher will give you better resolution, but will take longer. As for the beta diversity plots, you can open the resulting HTML file to view the plots: plots/alpha_rarefaction_plot/alpha_rarefaction_plots/rarefaction_plots.html

Choose "observed_otus" as the metric and "Source" as the category. You should see this plot:

![](https://www.dropbox.com/s/70twdntt1m3nnlk/alpha.png?raw=1)

There is no difference in the number of OTUs identified in the guts of mice from the BZ facility than the CJS facility, based on this dataset. However, since the rarefaction curves have not reached a plateau, it is likely that this comparison is just incorrect to make with so few reads. Indeed, [with the full dataset you do see a difference in the number of OTUs](https://github.com/mlangill/microbiome_helper/wiki/16S-tutorial-(long-running-time)).

### Testing for statistical differences   

So what's the next step? Since we know that source facility is such an important factor, we could analyze samples from each facility separately. This will lower our statistical power to detect a signal, but otherwise we cannot easily test for a difference between genotypes. 

To compare the genotypes within the two source facilities separately we fortunately don't need to re-run the OTU-picking. Instead, we can just take different subsets of samples from the final OTU table. First though we need to make two new mapping files with the samples we want to subset:

    head -n 1 map.txt >> map_BZ.txt; awk '{ if ( $3 == "BZ" ) { print $0 } }' map.txt >>map_BZ.txt
    head -n 1 map.txt >> map_CJS.txt; awk '{ if ( $3 == "CJS" ) { print $0 } }' map.txt >>map_CJS.txt

These commands are split into 2 parts (separated by ";"). The first part writes the header line to each new mapping file. The second part is an awk command that prints any line where the 3rd column equals the id of the source facility. Note that awk splits by any whitespace by default, which is why the source facility IDs are in the 3rd column according to awk, even though we know this isn't true when the file is tab-delimited.

The BIOM "subset-table" command requires a text file with 1 sample name per line, which we can generate by these quick bash commands:

    tail -n +2  map_BZ.txt | awk '{print $1}' > samples_BZ.txt
    tail -n +2  map_CJS.txt | awk '{print $1}' > samples_CJS.txt

These commands mean that the first line (the header) should be ignored and then the first column should be printed to a new file. We can now take the two subsets of samples from the BIOM file:

    biom subset-table -i final_otu_tables/otu_table.biom -a sample -s samples_BZ.txt -o final_otu_tables/otu_table_BZ.biom
    biom subset-table -i final_otu_tables/otu_table.biom -a sample -s samples_CJS.txt -o final_otu_tables/otu_table_CJS.biom

 We can now re-create the beta diversity plots for each subset:

    beta_diversity_through_plots.py -m map_BZ.txt -t clustering/rep_set.tre -i final_otu_tables/otu_table_BZ.biom -o plots/bdiv_otu_BZ 
    beta_diversity_through_plots.py -m map_CJS.txt -t clustering/rep_set.tre -i final_otu_tables/otu_table_CJS.biom -o plots/bdiv_otu_CJS 

We can now take a look at whether the genotypes separate in the re-generated weighted beta diversity PCoAs for each source facility separately.

For the BZ source facility:

![](https://www.dropbox.com/s/0xptjwuzikf7jfz/BZ_beta.png?raw=1)

And for the CJS source facility:

![](https://www.dropbox.com/s/by7d555syvv55qa/CJS_beta.png?raw=1)

Just by looking at these PCoA plots it's clear that if there is any difference it's subtle. To statistically evaluate whether the weighted UniFrac beta diversities differ among genotypes within each source facility, you can run an analysis of similarity (ANOSIM) test. These commands will run the ANOSIM test and change the output filename:

    compare_categories.py --method anosim -i plots/bdiv_otu_BZ/weighted_unifrac_dm.txt -m map_BZ.txt -c genotype -o beta_div_tests
    mv beta_div_tests/anosim_results.txt  beta_div_tests/anosim_results_BZ.txt 

    compare_categories.py --method anosim -i plots/bdiv_otu_CJS/weighted_unifrac_dm.txt -m map_CJS.txt -c genotype -o beta_div_tests
    mv beta_div_tests/anosim_results.txt  beta_div_tests/anosim_results_CJS.txt 

You can take a look at the output files to see significance values and test statistics. The P-values for both tests are > 0.05, so there is no significant difference in the UniFrac beta diversities of different genotypes within each source facility.

## Using STAMP to test for particular differences
  
Often we're interested in figuring out which particular taxa (or other feature such as functions) differs in relative abundance between groups. There are many ways this can be done, but one common method is to use the easy-to-use program [STAMP](http://kiwi.cs.dal.ca/Software/STAMP). We'll run STAMP on the full OTU table to figure out which genera differ between the two source facilities as an example.

Before running STAMP we need to convert our OTU table into a format that STAMP can read:
  
    biom_to_stamp.py -m taxonomy final_otu_tables/otu_table.biom >final_otu_tables/otu_table.spf

If you take a look at "final_otu_tables/otu_table.spf" with _less_ you'll see that it's just a simple tab-delimited table.

Now we're ready to open up STAMP, which you can either do by typing _STAMP_ on the command-line or by clicking the side-bar icon indicated below.  

<img src="https://www.dropbox.com/s/y6kneqwfwf8zvtx/STAMP_icon.png?raw=1" width="233.5" height="293.5">

To load data into STAMP click "File" and then "Load data..." as indicated below.

<img src="https://www.dropbox.com/s/ug2p7812pmgxdv3/STAMP_load_data.png?raw=1" width="267.5" height="191">

Load "otu_table.spf" as the Profile file and "map.txt" as the Group metadata file. 

As a reminder, the full paths of these files should be: /home/mh_user/Desktop/16S_chemerin_tutorial/final_otu_tables/otu_table.spf and /home/mh_user/Desktop/16S_chemerin_tutorial/map.txt

Change the Group field to "Source" and the profile level to "Level_6" (which corresponds to the genus level). Change the heading from "Multiple groups" to "Two groups". The statistical test to "Welch's t-test" and the multiple test correction to "Benjamini-Hochberg FDR" (as shown below).  

<img src="https://www.dropbox.com/s/5fv7hcr5rx5eb2k/STAMP_settings.png?raw=1" width="242.5" height="328.5">

Change the plot type to "Bar plot".

**Q9)** Look at the barplot for Prevotella and save it to a file.

You can see how many genera are significant by clicking "Show only active features".
