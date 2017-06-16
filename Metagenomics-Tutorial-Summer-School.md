**THIS PAGE IS CURRENTLY UNDER CONSTRUCTION **

# Introduction

This tutorial is set-up to walk you through the process of determining the taxonomic and functional composition of several metagenomic samples. It covers the use of [Metaphlan2](http://huttenhower.sph.harvard.edu/metaphlan2) (for taxonomic classification), [HUMAnN2](http://huttenhower.sph.harvard.edu/humann2) (for functional classification), and [STAMP](http://kiwi.cs.dal.ca/Software/STAMP) (for visualization and statistical evaluation).

Throughout the tutorial, there are several questions to ensure that you are understanding the process (and not just copy and pasting). You can check the [answer sheet](https://github.com/mlangill/microbiome_helper/wiki/Metagenomics-Summer-School-Tutorial-Answers) after you have answered them yourself.

We'll be using a subsampled version of the metagenomics dataset from [Schmidt et al. (PLoS ONE 2014)](http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0098741) that investigated changes in the oral microbiome associated with oral cancers. **Note: since these files have been drastically subsampled they do not represent typical metagenomics raw files.** For instance, many more taxonomic and functional classifications would be output if the full datasets were used. However, due to the shallow read depth the running time of the workflow is extremely fast! 

**Authors**: Morgan Langille and Gavin Douglas

**First Created**: June 2017

## Requirements
* Basic unix skills (there are many tutorials online such as [this one](http://korflab.ucdavis.edu/bootcamp.html))  
* [Tutorial Data]()

## Initial Setup
Download the tutorial data, save it to the Desktop (within Ubuntu), unzip the folder, and enter this folder. You can do this using the below commands.  
  
Open a terminal/console and change to the directory containing the tutorial data:  
(paste the below command with your mouse, you probably wont be able to copy/paste text in the VBox with keyboard commands)  

```
cd ~/Desktop/  
wget http://kronos.pharmacology.dal.ca/public_files/tutorial_datasets/mgs_tutorial.zip
unzip mgs_tutorial.zip   
cd mgs_tutorial  
```

### Explore Samples  
  
The first step in any metagenomics pipeline should be to explore your raw files. For this tutorial there should be a "mapping file", which contains information about each of the samples called **map.txt**, along with the sequence data for each sample contained within separate FASTQ files within the directory **subsampled_fastq**.

Lets take a look at the **map.txt** using _less_ (type q to quit out of the program when done)

```
less map.txt
```

You will have noticed that this map file contains three columns with the first being sample ids, the middle column being the body site, and the third being the sex of the person. 

Remember you can count the lines of a file using _wc -l_. For example,

```
wc -l map.txt
```

**Q1)** How many samples are there in total (you can either look at the map.txt file or count the FASTQ files)?

**Q2)** How many samples are there of each sample type?

Before continuing let's take a look at the raw sequence files themselves. Typically it's better to keep these files in gzipped format to save disk space. However, since several commands below require the files to be uncompressed we can gunzip them now:

```
gunzip subsampled_fastqs/*gz
```

[FASTQ format](https://en.wikipedia.org/wiki/FASTQ_format) is currently the most common format for raw sequencing reads. In this format, information for each read is split over 4 lines: a header (i.e. the read name and other details), the sequence, a line containing "+", and the quality scores for each nucleotide.  
  
To visualize this we can look at the first 8 lines of one of the FASTQs using the _head_ command:

```
head -n 8 subsampled_fastqs/p136C_R1.fastq

@SRR3586062.883556
CTTGGGGCTGCTGAGCTTCATGCTCCCCTCCTGCCTCAAGGACAATAAGGAGATCTTCGACAAGCCTGCAGCAGCTCGCATCGACGCCCTCATCGCTGAGG
+
CCCFFFFFHHHHHIJJJJJJJIJIJJJJGIJDGIJEIIJIJJJJJJJJIJJJJIJJIJJJJJHHHFFFFECEEEDDDDD?BDDDDDDBDDDDDDDDBBBDD
@SRR3586062.3376311
GACGGTGTCCTCAGGACCCTTCAGTGCCTTCATGATCTGCTCAGAGGTGATGGAGTCACGGACGAGATTCGTCGTGTCAGCACGTAGGATGCGGTCGCCTG
+
@@@DDDDAFF?DF;EH+ACHIIICHDEHGIGBFE@GCGDGG?D?G@BGHG@FHCGC;CC:;8ABH>BECCBCB>;8ABCCC@A@#################
```

**Q3)** To save time each sample has been subsampled to a relatively low number of reads. How many sequences are there in each FASTQ? 

For your own data you may identify outlier samples with low read depth at this stage. It would also be advisable to explore the quality of your data with a tool like [FASTQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/). 

## Pre-processing  

The strength of shotgun metagenomics is also its weakness: all DNA in your samples will be sequenced with similar efficiencies. This is a problem when host microbiome samples are taken since it's possible to get substantial amounts of host DNA included in your raw sequences. You should screen out these contaminant sequences before running taxonomic and functional classification. It's also a good idea to screen out PhiX sequences in your data: this virus is a common sequencing control since it has such a small genome. 

Below we will screen out reads that map to the human and/or PhiX genomes. If your samples were taken from a different host you will need to map your reads to that genome instead. 

```
echo "--very-sensitive-local" >> ./bowtie2_config.txt

run_contaminant_filter.pl -p 4 -o screened_reads/ subsampled_fastqs/* -d /home/shared/bowtiedb/GRCh38_PhiX -c ./bowtie2_config.txt 
```
The numbers and percentages of reads removed from each FASTQ are reported in the _screened\_reads.log_ file by default.

**Q4)** Which sample had the highest number of reads screened out?

Next it's a good idea to run quality filtering of your samples. [Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic) is one popular tool for filtering and trimming next-generation sequencing data. Typically you would first explore the quality of your data to choose sensible cut-offs using FASTQC or a similar alternative. The below command will run trimmomatic on each sample (both forward and reverse reads at once) and use some typical quality cut-offs. Note that if you have overlapping reads you might want to stitch them together first using [PEAR](https://sco.h-its.org/exelixis/web/software/pear/).

```
run_trimmomatic.pl -l 5 -t 5 -r 15 -w 4 -m 70 -j /usr/local/prg/Trimmomatic-0.36/trimmomatic-0.36.jar --thread 4 -o trimmomatic_filtered screened_reads/*fastq 
```

To see what each option means you can type _run\_trimmomatic.pl -h_, which will output this description for the above options:

```
    -o, --out_dir <file>
        Output directory for filtered fastq files (default:
        "trimmomatic_filtered").

    --thread <# of CPUs>
        Number of CPUs to thread each job on (default: 1).

    -l,--leading_quality <int>
        The minimum quality for leading bases to be kept as required by
        Trimmomatic's LEADING command (default: 5).

    -t,--trailing_quality <int>
        The minimum quality for trailing bases to be kept as required by
        Trimmomatic's TRAILING command (default: 5).

    -r,--required_quality <int>
        Average quality required by Trimmomatic's SLIDINGWINDOW command
        (default: 15).

    -w,--window_size <int>
        Window sizes of sliding windows required by Trimmomatic's
        SLIDINGWINDOW command (default: 4).

    -m,--min_length <int>
        Min length of reads to be retained as required by Trimmomatic's MINLEN
        command (default: 70).

    -j,--jar <jarfile>
        Path to Trimmomatic jarfile (default:
        /usr/local/prg/Trimmomatic-0.36/trimmomatic-0.36.jar).
```
The counts and percentages of reads dropped is reported in _trimmomatic\_tabular\_log.txt_ by default. Only read pairs that both passed the quality thresholds will be retained for downstream steps.    

Lastly, since we didn't stitch the paired-end reads together at the beginning of this workflow we will concatenate the FASTQs together now before running Metaphlan2 and HUMAnN2 since these programs [do not use paired-end information](https://bitbucket.org/biobakery/humann2/wiki/Home#markdown-header-humann2-and-paired-end-sequencing-data). 

```
concat_paired_end.pl -p 4 -o cat_reads trimmomatic_filtered/*_paired*fastq 
```

## Taxonomic and Functional Profiling with Metaphlan2 and HUMAnN2 Respectively

Metaphlan2 is a standalone tool and can be run with the _metaphlan2.py_ command. It is easier to run a custom analysis with this script directly. However, since HUMAnN2 runs this script anyway (with default options unless specified otherwise) we can use the output files produced by this pipeline instead, which is simpler! 

To begin with make sure the program _humann2_ is in your PATH. This program is frequently updated so it's a good idea to check what version you're running.

**Q5)** What version of HUMAnN2 are you using? What version of metaphlan2.py?

### Running the HUMAnN2 Pipeline (which includes Metaphlan2)

Both humann2 and metaphlan2.py come with a large number of options which could be useful (take a look by running each program with the _-h_ option). Since humann2 can take a while to run we'll just run one sample for this tutorial (key output files are in the _precalculated_ folder already).

```
humann2 --threads 4 --input cat_reads/p144C.fastq  --output humann2_out/
```

You should see a log of what HUMAnN2 is doing printed to screen:

```
Running metaphlan2.py ........

Found g__Fusobacterium.s__Fusobacterium_nucleatum : 22.70% of mapped reads
Found g__Neisseria.s__Neisseria_unclassified : 18.81% of mapped reads
Found g__Fusobacterium.s__Fusobacterium_periodonticum : 10.78% of mapped reads
Found g__Prevotella.s__Prevotella_melaninogenica : 10.48% of mapped reads
Found g__Haemophilus.s__Haemophilus_parainfluenzae : 9.52% of mapped reads
Found g__Gemella.s__Gemella_unclassified : 5.48% of mapped reads
Found g__Veillonella.s__Veillonella_unclassified : 3.84% of mapped reads
Found g__Sphingomonas.s__Sphingomonas_melonis : 2.01% of mapped reads
Found g__Gemella.s__Gemella_haemolysans : 1.16% of mapped reads

Total species selected from prescreen: 9
```

The Metaphlan2 output file is the "*bugs_list.tsv" in the temp folder within the output folder. You can take a look at it with _less_. 

Your should see something like this:

```
#SampleID       Metaphlan2_Analysis
k__Bacteria     100.0
k__Bacteria|p__Fusobacteria     48.69306
k__Bacteria|p__Proteobacteria   30.34017
k__Bacteria|p__Firmicutes       10.48718
k__Bacteria|p__Bacteroidetes    10.47959
k__Bacteria|p__Fusobacteria|c__Fusobacteriia    48.69306
k__Bacteria|p__Proteobacteria|c__Betaproteobacteria     18.80684
k__Bacteria|p__Bacteroidetes|c__Bacteroidia     10.47959
k__Bacteria|p__Proteobacteria|c__Gammaproteobacteria    9.51996
k__Bacteria|p__Firmicutes|c__Bacilli    6.64301
k__Bacteria|p__Firmicutes|c__Negativicutes      3.84417
```

You can see that the output contains two columns, with the first column being the taxonomy name and the second column representing the relative abundance of that taxa (out of 100 total).

This output file provides summaries at each taxonomic level (e.g. phylum, class, family, genus, species, and sometimes strain level). At each taxonomic level the relative abundances will sum to 100. 

If reads can only be assigned to a certain taxonomic level (e.g. say the family level), then Metaphlan2 will put those unassigned reads into a lower level taxonomic rank with the name "unclassified" appended.

For example lets pull out all the reads assigned to the genus _Gemella_ using the _grep_ command:

    grep g__Gemella p144C_metaphlan_bugs_list.tsv
 
You should get output like this:

```
k__Bacteria|p__Firmicutes|c__Bacilli|o__Bacillales|f__Bacillales_noname|g__Gemella	6.64301
k__Bacteria|p__Firmicutes|c__Bacilli|o__Bacillales|f__Bacillales_noname|g__Gemella|s__Gemella_unclassified	5.48167
k__Bacteria|p__Firmicutes|c__Bacilli|o__Bacillales|f__Bacillales_noname|g__Gemella|s__Gemella_haemolysans	1.16134
k__Bacteria|p__Firmicutes|c__Bacilli|o__Bacillales|f__Bacillales_noname|g__Gemella|s__Gemella_haemolysans|t__Gemella_haemolysans_unclassified	1.16134
```

You can see that 6.64% of the metagenome is predicted from organisms in the genus _Gemella_. Of those 1.16% are assigned to the species _Gemella haemolysans_, while 5.48% are assigned to s__Gemella_unclassified, which means Metaphlan2 doesn't know what species to actually call them. 

**Q6)** What is the relative abundance of organisms unclassified at the species level for the genus _Neisseria_? (Remember to use the grep command)

### Running HUMAnN2 on all FASTQs with GNU Parallel
**You don't need to do this for the tutorial**, but the below command could be used to run HUMAnN2 on each FASTQ using default options with [GNU parallel](https://www.gnu.org/software/parallel/) to loop over each file. You can see [our tutorial](https://github.com/mlangill/microbiome_helper/wiki/Quick-Introduction-to-GNU-Parallel) for more details on this command, but briefly:
* _-j_ indicates the number of jobs to run.
* _--eta_ outputs the expected time to finish as individual jobs end.
* The files being looped over follow _:::_.
* Everything within the single-quotes (') is the actual command being run on every file. The options within the quotes are for the particular tool being run and NOT parallel.
* {} specifies each individual filename being looped over.
* {/.} specifies the individual filename being looped over with the extension and PATH removed.

**It's a good idea to run parallel with the _--dry-run_ option** the first time you are running a set of files. This option will echo the commands that would have been run to screen without running them. This can be very helpful for troubleshooting parallel syntax errors! 

```
parallel -j 1 'humann2 --threads 4 --input {} --output humann2_out/' ::: cat_reads/*fastq
```

Setting the option _--memory-use maximum_ will speed up the program **if you have enough available memory**.

### Merging Metaphlan2 Results 

We've placed all of the Metaphlan2 output files in _precalculated/metaphlan2_out_. All of these files can be merged into one table with this command:

```
/usr/local/prg/metaphlan2/utils/merge_metaphlan_tables.py precalculated/metaphlan2_out/*tsv > etaphlan2_merged.tsv
```

Note that this script 'merge_metaphlan_tables.py' takes one or more Metaphlan2 output files as input and combines them into a single output file. 

The merged output file should look like this:
```
ID      p136C_metaphlan_bugs_list       p136N_metaphlan_bugs_list       p143C_metaphlan_bugs_list       p143N_metaphlan_bugs_list       p144C_metaphlan_bugs_list       p144N_metaphlan_bugs_list       p153C_metaphlan_bugs_list       p153N_metaphlan_bugs_list       p156C_metaphlan_bugs_list       p156N_metaphlan_bugs_list
#SampleID       Metaphlan2_Analysis     Metaphlan2_Analysis     Metaphlan2_Analysis     Metaphlan2_Analysis     Metaphlan2_Analysis     Metaphlan2_Analysis     Metaphlan2_Analysis     Metaphlan2_Analysis     Metaphlan2_Analysis     Metaphlan2_Analysis
k__Bacteria     100.0   100.0   100.0   100.0   100.0   100.0   100.0   100.0   100.0   100.0
k__Bacteria|p__Actinobacteria   0.10365 0.72239 0.0     29.62481        0.0     13.21533        0.0     30.90135        0.36794 6.80878
k__Bacteria|p__Actinobacteria|c__Actinobacteria 0.10365 0.72239 0.0     29.62481        0.0     13.21533        0.0     30.90135        0.36794 6.80878
k__Bacteria|p__Actinobacteria|c__Actinobacteria|o__Actinomycetales      0.0     0.50942 0.0     29.62481        0.0     13.21533        0.0     30.90135        0.36794 6.80878
k__Bacteria|p__Actinobacteria|c__Actinobacteria|o__Actinomycetales|f__Actinomycetaceae  0.0     0.0     0.0     0.22061 0.0     0.0     0.0     0.0     0.0     0.0
k__Bacteria|p__Actinobacteria|c__Actinobacteria|o__Actinomycetales|f__Actinomycetaceae|g__Actinomyces   0.0     0.0     0.0     0.22061 0.0     0.0     0.0     0.0     0.0     0.0
```
To read this table into [STAMP](http://kiwi.cs.dal.ca/Software/STAMP) we'll need to convert it to unstratified format with this command:

```

Now each sample is listed as a different column within this output file. You can view this file again using 'less' or you can import it into your favourite spreadsheet program.

### Analyzing Metaphlan2 output with STAMP

STAMP takes two main files as input the profile data which is a table that contains the abundance of features (i.e. taxonomic or functions) and a group metadata file which provides more information about each of the samples in the profile data file. 

The metadata file is the **hmp_map.txt** file. This file is present already in the "hmp_metagenomics_downsampled" directory.
You will also need the profile data file which is in a different format than what Metaphlan2 outputs. Therefore we first need to convert it using a script from the microbiome_helper package. You can convert the pre-computed Metaphlan2 output for all samples like this:

    metaphlan_to_stamp.pl pre-computed_results/metaphlan2_output/metaphlan_merged_all.txt > metaphlan_merged_all.spf

As you can see this file takes the Metaphlan2 output file (from all the merged samples), which we already made for you, as the only parameter and then writes the output to a new file called "metaphlan_merged_all.spf".

Now load the profile file (metaphlan_merged_all.spf and the metadata file (hmp_map.txt) files by going to "File->load data" within STAMP:

![](https://www.dropbox.com/s/o20hvb0m2fsgcyq/STAMP_File_Menu.png?raw=1)

Change the “Profile level” (top left) to “Genus”, ensure that the Group legend (top right) has been set to “sex”, and that “PCA plot” has been set below the large middle window. 

You should now be looking at a PCA plot that is coloured by the samples being male or female like this:

![](https://www.dropbox.com/s/hs3au04ftlsofyb/sex_PCA_taxonomy_downsampled_redone.png?raw=1)

As you can see there is no obvious separation in the data points when coloured by sex of the person giving the sample. 

Now change the group field (top right) to “body_site” and the PCA will be coloured according to that grouping instead.  

**Q10)** Do you see any separation in the samples when coloured by body site? If so, describe the body sites that separate and which PC axis differentiates these samples.

Now lets test what is significantly different between the groups at the Genus rank. 

Under the “Multiple groups” dialog on the left, check that ANOVA is being used as the statistical test, and select “Benjamini-Hochberg FDR” for the multiple test correction. The box at bottom will say how the “Number of active features” using these set of statistics and should indicate that there are 3 organisms at the genus level that are statistically different in their means across the three body sites. 

<img src="https://www.dropbox.com/s/6xxroa5z4jwno5s/taxonomy_bodytype_downsampled_setup.png?raw=1" alt="body_site config" width="526" height="600">

Now lets look at visualizations of these different taxonomic groups. 

Change the drop-down box that says "PCA plot" to "Bar plot".

A new window will show a list of all the Genera. You can filter this list to just the 3 Genera passing the statistical test by clicking the "Show only active features" box below the list of Genera. 

You can also order the list of Genera by the most significant by clicking on the "corrected p-value" header. Now if you click on the feature "g_Granulicatella" you should see a bar chart representing the relative abundance for each sample for this Genera along with the mean value for the group indicated by a horizontal line. The plot should look like this:

<img src="https://www.dropbox.com/s/q9bwtdse5q33ods/graniculata_downsampled.png?raw=1" alt="graniculata" width="500" height="500">

Explore each of the different visualizations by changing "Bar plot" to each of the other options available (e.g. Heatmap plot, Post-hoc plot, and Box plot).

**Q11)** Create a box plot for the family Streptococcaceae (Change "Profile Level" to "Family" in the top left). Save the box plot as a .png image by using the File->Save plot feature in STAMP.

You can exit STAMP once you're done with this section.

***

### STAMP with HUMAnN Output

Load the kos.spf file along with the original hmp_map.txt file into STAMP. (File->Load)

Click on the "Two Groups" tab and choose "Benjamini-Hochberg FDR" for Multiple Test Correction. Look at the "Number of active features" to determine if there are any significant different features.

**Q2)** Using a two group test with multiple test correction applied are there any significant differences between male and female samples?

Change to a Multiple Group Test using the same multiple test correction and change the Group field (top right) to "body_site".

**Q3)** Are there any significant differences in KOs across the body sites? If so, how many?

Now load the "pathways.spf" file into STAMP using the same "hmp_map.txt" file. 

(Note: STAMP has a bug that will not load a new dataset on top of another, so you need to completely close and restart STAMP first before loading in a different dataset)

**Q4)** When comparing samples by body site, how many KEGG Pathways are significantly different when using a group test (ANOVA) with Benjamini-Hochberg FDR?

Choose "Box plot" instead of PCA plot, and then expand the list of KEGG Pathways so you can see the corrected p-value. Order the list by corrected p-value by clicking on that column header. 

**Q5)** What is the most significantly different KEGG Pathway? What is the corrected p-value for this KEGG Pathway?

Compare the tongue samples to all other samples using a Two Group test. First select “tongue_dorsum” for Group 1 (on the left hand side) and then select “All other samples” as Group 2. Use the default Welch’s t-test with BH FDR.

Your PCA plot should look like this:

![](https://www.dropbox.com/s/ljdk2o39hopjlfe/functional_tongue_vs_others_downsampled.png?raw=1)

**Q6)** How many KEGG Pathways are significantly different between the tongue and the plaque samples combined?

Create a "Bar plot" for "ko00230: Purine metabolism".

**Q7)** Is the relative abundance of “Purine metabolism” higher or lower in the tongue compared to the plaque samples?

**Q8)** Create an “Extended error bar” plot and save the image as a .png using the File->Save Plot option.