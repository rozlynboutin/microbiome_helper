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
/usr/local/prg/metaphlan2/utils/merge_metaphlan_tables.py precalculated/metaphlan2_out/*tsv > metaphlan2_merged.tsv
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
metaphlan_to_stamp.pl metaphlan2_merged.tsv > metaphlan2_merged.spf
```

Finally, notice that the sample names in _map.txt_ don't match the column names in _metaphlan2_merged.spf_. These need to match for STAMP to read the table. This can be fixed by removing all instances of "_metaphlan_bugs_list".

```
sed -i 's/_metaphlan_bugs_list//g' metaphlan2_merged.spf 
```

Now each sample is listed as a different column within this output file. You can view this file again using 'less' or you can import it into your favourite spreadsheet program.

### Analyzing Metaphlan2 output with STAMP

STAMP takes two main files as input the profile data which is a table that contains the abundance of features (i.e. taxonomic or functions) and a group metadata file which provides more information about each of the samples in the profile data file. 

The metadata file is the **map.txt** file. This file is present already in the _mgs\_tutorial_ directory.
You will also need the profile data file that we just created. 

Load the profile file (metaphlan2_merged.spf and the metadata file (map.txt) files by going to "File->load data" within STAMP:

![](https://www.dropbox.com/s/o20hvb0m2fsgcyq/STAMP_File_Menu.png?raw=1)

You may have noticed that a "Cancer" and "Normal" samples was taken from the same individuals. A pertinent question is whether these profiles are more similar simply because they are from the same individual. To investigate this, change the “Profile level” (top left) to “Genus”, ensure that the Group legend (top right) has been set to _Individual_, and that “PCA plot” has been set below the large middle window. 

You should now be looking at a PCA plot that is coloured by which individual the samples came from:

![]

As you can see there is no obvious separation in the data points when coloured by individual - at the very least this isn't driving the major axes of variation. 

Now change the group field (top right) to “Sample Type” and the PCA will be coloured according to that grouping instead.  

**Q7)** How much of the variation is explained by PC3?

Now lets test what is significantly different between the groups at the Genus rank. 

Under the “Two groups” dialog on the left, check that Welch's t-test is being used as the statistical test, and select “Benjamini-Hochberg FDR” for the multiple test correction. 

![]

No genera are significantly different between samples based on sample type. This is likely at least partially due to the low sample sizes and low sequencing depths used for this tutorial. For the purposes of this tutorial we will focus on one genus that was significant before FDR correction. If we were to report this finding elsewhere we would need to clearly state the caveat that this genus was not significant after multiple-test corrections.

**Q8)** Which genus is significant based on its _raw_ P-value?

Now lets look at a visualization of this genus.

Change the drop-down box that says "PCA plot" to "Box plot".

A new window will show a list of all the Genera. Select just the genus of interest to plot its relative abundance.

**Q9)** Save this boxplot by going to File -> Save plot...

In the next section we'll be reading in the HUMAnN2 output into STAMP - it's easier to close and reopen STAMP when reading in new files.

### Analyzing a Subset of the HUMAnN2 output 

Similar analyses can be run on the pathway abundances identified with HUMAnN2 as above. To begin with we can join all individual tables into a single table as above.

```
humann2_join_tables --input precalculated/humann2_out/ --file_name pathabundance --output humann2_pathabundance.tsv
```

Then we'll want to normalize each sample into relative abundance (so that the counts for each sample sum to 100).

```
humann2_renorm_table --input humann2_pathabundance.tsv --units relab --output humann2_pathabundance_relab.tsv
```

We could read this entire table (after running the reformatting commands below) and run similar analyses as we did above.

However, since HUMAnN2 yields functions stratified by taxa we could limit ourselves just to those functions that were found within the genus Streptococcus in order to focus on the above finding with the Metaphlan2 data. You could once again use _grep_ to slice out only those pathways that are linked to this genus specifically.

In addition, more focused analyses could be based off of prior knowledge. For instance, pathways related to sugar degradation are especially of interest in the oral microbiome due to its relationship with dental health. We will test for differences between cancer and normal samples in STAMP similar to the earlier analysis based on this approach.

First an unstratified version of the pathway abundance table needs to be generated (the below command will also create a stratified version only in the same directory):

```
humann2_split_stratified_table --input humann2_pathabundance_relab.tsv --output ./
```

Then we can parse out the headerline and any lines matching the pathway of interest (LACTOSECAT-PWY: lactose and galactose degradation I).

```
head -n 1 humann2_pathabundance_relab_unstratified.tsv >> humann2_pathabundance_relab_LACTOSECAT-PWY.tsv
grep "LACTOSECAT-PWY" humann2_pathabundance_relab_unstratified.tsv >> humann2_pathabundance_relab_LACTOSECAT-PWY.tsv
```

Using _sed_ to make string replacements we can also format the header-line for input into STAMP.

```
sed 's/_Abundance//g' humann2_pathabundance_relab_LACTOSECAT-PWY.tsv >  humann2_pathabundance_relab_LACTOSECAT-PWY.spf
sed -i 's/# Pathway/MetaCyc_pathway/' humann2_pathabundance_relab_LACTOSECAT-PWY.spf
```

Read file into STAMP as before with the mapping file. 

**Q10)** Run a two-sided Welch's t-test  between sample types for this pathway. What is the P-value?