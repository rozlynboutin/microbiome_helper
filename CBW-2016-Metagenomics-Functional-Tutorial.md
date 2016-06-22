  * [Introduction](#introduction)
    * [Requirements](#requirements)
    * [Initial Setup](#initial-setup)
    * [Explore Samples](#explore-samples)
  * [Determining Functional Composition with HUMAnN](#determining-functional-composition-with-humann)
    * [Running DIAMOND search against KEGG](#running-diamond-search-against-kegg)
    * [Running HUMAnN](#running-humann)
    * [Running all samples with Microbiome Helper](#running-all-samples-with-microbiome-helper)
    * [STAMP with HUMAnN Output](#stamp-with-humann-output)

# Introduction

This tutorial is set-up to walk you through the process of determining the functional composition of several metagenomic samples. It covers the use of [HUMAnN](https://huttenhower.sph.harvard.edu/humann) and [STAMP](http://kiwi.cs.dal.ca/Software/STAMP) (for visualization and statistical evaluation).  

Throughout the tutorial, there are several questions to ensure that you are understanding the process (and not just copy and pasting). You can check the [answer sheet](CBW-2016-Metagenomics-Tutorial-Answers) after you have answered them yourself.

This lab component will use samples collected and sequenced through the Human Microbiome Project (HMP).  We will be analyzing samples collected from three oral sites including subgingival, supragingival, and the tongue. These samples have been downloaded for you already from the HMP-DACC (http://hmpdacc.org/HMIWGS/all/) and have been randomly subsampled to a much smaller number of reads to allow faster computation time. Real data will require more waiting time between each step!

**Author**: Morgan Langille

**Contributions by**: Gavin Douglas

**First Created**: Summer 2015 for CBW Metagenomics, Halifax

**Last Edited**: Summer 2016 for CBW Metagenomics, Vancouver

## Requirements

* Basic unix skills (This is a good introductory tutorial: http://korflab.ucdavis.edu/bootcamp.html)

* These samples have been downloaded for you already from the HMP-DACC (http://hmpdacc.org/HMIWGS/all/) and placed in the directory: ~/CourseData/metagenomics/hmp_metagenomics/

## Initial Setup

Prepare working directory:

    rm -rf ~/workspace/module_function
    mkdir -p ~/workspace/module_function
    cd ~/workspace/module_function
    ln -s ~/CourseData/metagenomics/hmp_metagenomics ./

*Notes*:

* The `ln -s` command adds a symbolic link to the (read-only) `~/CourseData/metagenomics/hmp_metagenomics` directory.

### Running DIAMOND search against KEGG database

Before running HUMAnN, it is first necessary run a similarity search against the KEGG database. This database is a collection of pathways for metabolic and other functional molecular and cellular processes, and is located at “~/CourseData/metagenomics/ref/kegg”. This is a reduced KEGG database that the authors of HUMAnN created and which they make available through personal correspondence. This similarity search can be done using various tools such as BLAST, USEARCH, RapSearch, etc. In this tutorial we will use a relatively new similarity search tool that is much faster and works well for large metagenomic datasets called [DIAMOND](http://ab.inf.uni-tuebingen.de/software/diamond/).

First we will make a directory to store our sequence search outputs:

    mkdir pre_humann

Now we will run DIAMOND on our metagenomic sequences for the sample SRS015044:
 
    diamond blastx -p 8 -d ~/CourseData/metagenomics/ref/kegg/kegg.reduced -q ./hmp_metagenomics/fastq/SRS015044.fastq -a pre_humann/SRS015044

The options used in this command are:

* 'blastx': Tells DIAMOND to run in “blastx” mode meaning that we will search a nucleotide query against a protein database in all 6 frame translations (3 forward and 3 reverse).
* '-p 8': Indicates that DIAMOND should use 8 threads to do the search.
*  '-d kegg/kegg.reduced' points at the KEGG database which has already been formatted for use with DIAMOND.
* '-q ./hmp_metagenomics/fastq/SRS015044.fastq' is the input metagenomic sample
* '-a pre_humann/SRS015044' is the name of the output file (Note: DIAMOND appends a '.daa' automatically to the end of the output file name)
* Note: That the default e-value cutoff used by DIAMOND is 0.001.

The previous command is comparing 60k sequences vs. the KEGG database (~1.3 million sequences) and takes about <2 minutes with 8 CPUs. If we had used the NCBI BLASTX it would have taken a couple of days!

The  output from DIAMOND is a special binary format that we can then turn into either SAM or BLAST tabular output with the latter being the default.

    diamond view -a pre_humann/SRS015044.daa -o pre_humann/SRS015044.txt

You can see the output using _less_:

    less pre_humann/SRS015044.txt

Each column tells us different information about the match:

1.	Query  
2.	Subject  
3.	% id  
4.	alignment length  
5.	Mistmatches  
6.	gap openings  
7.	q.start  
8.	q.end  
9.	s.start  
10.	s.end  
11.	e-value  
12.	bit score  

Now run DIAMOND for samples SRS015893 and SRS097871 against the KEGG database and convert the output to BLAST tabular format as above. **(You need to figure out the commands to run and execute them before moving to the next step)**

### Running HUMAnN

Now, we are going to run HUMANN. To use human you are going to copy the entire program to your working directory. The human program is in “ ~/CourseData/refs/humann-0.99/”:

    cp -r ~/CourseData/metagenomics/ref/humann-0.99/ ./

Then move into the humann directory, ensure the scripts in /src are executable, and lastly remove the "mock" sample in the input directory:

    cd humann-0.99
    chmod u+x src/*
    rm input/mock_even_lc.txt.gz

Now copy your blast output from your 3 samples into the “human-0.99/input” directory:

    cp ../pre_humann/SRS015044.txt ../pre_humann/SRS015893.txt ../pre_humann/SRS097871.txt ./input/

To being running human on just these 3 samples you use the “scons” command (using 4 cores):

    scons -j 4

A bunch of messages will pass on your screen and it should finish in ~2 minutes. All of the output is contained in the “humann-0.99/output” directory. To see a list of them type:
 
    ls output

There are MANY output files from HUMAnN. The ones we care about are called:

* **01b-hit-keg-cat.txt** –> KEGG KOs
* **04b-hit-keg-mpm-cop-nul-nve-nve.txt** -> KEGG Modules
* **04b-hit-keg-mpt-cop-nul-nve-nve.txt** -> KEGG Pathways

These files contain relative abundances for each of these different functional classifications. You can look at the format of these using _less_. For example to look at the KEGG Pathways:

    less output/04b-hit-keg-mpt-cop-nul-nve-nve.txt

The output should look like this:

```
ID      NAME    SRS015044-hit-keg-mpt-cop-nul-nve-nve   SRS015893-hit-keg-mpt-cop-nul-nve-nve   SRS097871-hit-keg-mpt-cop-nul-nve-nve   mock_even_lc-hit-keg-mpt-cop-nul-nve-nve
RANDSID RANDSID 763901136       764305738                
START   START   Q1_2009 Q2_2009          
GENDER  GENDER  female  male             
VISNO   VISNO   1       1                
STSite  STSite  Supragingival_plaque    Tongue_dorsum            
Parent_Specimen Parent_Specimen 700023699       700024548                
Run ID  Run ID  620DH   61NTL            
Lane    Lane    7       7                
SRS     SRS     700023699       700024548                
Mean Quality    Mean Quality            32.57            
Number of Quality Bases Number of Quality Bases         4175865122               
Percent of Human Reads  Percent of Human Reads          0.0348           
Unique Non-Human Bases  Unique Non-Human Bases          4359898255               
InverseSimpson  InverseSimpson  59.8877 58.4796 44.4316 73.7574
Shannon Shannon 4.24908 4.22752 4.01834 4.41906
Pielou  Pielou  0.866226        0.861831        0.819188        0.90088
Richness        Richness        1       1       1       1
ko00564 ko00564: Glycerophospholipid metabolism 0.00878166      0.00639795      0.00925108      0.0094699
ko00680 ko00680: Methane metabolism     0.00521639      0.00556563      0       0.00440135
ko00562 ko00562: Inositol phosphate metabolism  0.00343186      0.00293449      0.00646182      0.0027541
ko00561 ko00561: Glycerolipid metabolism        0.00525597      0.0045911       0       0.00712117
```

* The first line indicates that the first column is a short id for the KEGG Pathway, the second column is a longer description of the KEGG Pathway, and the remaining columns represent the sample identifiers. 
* The next 13 lines contain metadata for each sample.
* The following 4 lines (starting at "InverseSimpson") calculate different alpha-diversity metrics for each sample, but in general these are usually not that useful and can be ignored.
* From line 19 onward, each row indicates the different relative abundance for each KEGG Pathway. Note that these values are small because they have been normalized so that each sample will sum to 1. 
* It should be noted that whether metadata rows are included or not depends on your settings.

Now, use less to look at the KO predictions and the KEGG Module predictions:

    less output/01b-hit-keg-cat.txt
    less output/04b-hit-keg-mpm-cop-nul-nve-nve.txt

**Q1)** Which of the three samples has the highest relative abundance of the KEGG Module: “M00319: Manganese/zinc/iron transport system”?

### Running all samples with Microbiome Helper

So what about all of our samples? To do the DIAMOND searches for all 30 you could use a wrapper script provided by the microbiome_helper package called run_pre_humann.pl. All 30 samples _could be_ processed with a single command like:  

    run_pre_humann.pl -p 8 -d ~/CourseData/metagenomics/ref/kegg/kegg.reduced -o pre_humann ./hmp_metagenomics/fastq/*

However, this would take awhile (30-60 min.)

To make things easier the output for all 30 samples has been pre-computed and is located in  “./hmp_metagenomics/pre-computed_results/humann_output”. 

If you browse the output using _less_ you can see that they are in the same format but with 30 columns representing the 30 samples:

    less ./hmp_metagenomics/pre-computed_results/humann_output/04b-hit-keg-mpt-cop-nul-nve-nve.txt

You will notice that the output looks fairly messy because the terminal will automatically line wrap and it becomes hard to see where one line ends and the next begins. I often find the "cut" command useful to browse data like this. "cut" allows you to just look at particular columns from the data. For example:

    cut -f 1-5 ./hmp_metagenomics/pre-computed_results/humann_output/04b-hit-keg-mpt-cop-nul-nve-nve.txt | less

* '-f 1-5' indicates that we want to look at the first 5 columns. We could just have easily used '-f 1,3,10' (to view columns 1, 3 and 10) or '-f 1-3,20-' (to view columns 1 to 3 and columns 20 onwards).
* '| less' "pipes" the output from the "cut" command and lets us view the output one screen at a time using our 'less' tool.  

Now, we are going to use STAMP to visualize the humann output just like we did with the taxonomic data. 

First, we need to convert the HUMAnN output files file into STAMP format:

    humann_to_stamp.pl ./hmp_metagenomics/pre-computed_results/humann_output/04b-hit-keg-mpt-cop-nul-nve-nve.txt > pathways.spf
    humann_to_stamp.pl ./hmp_metagenomics/pre-computed_results/humann_output/04b-hit-keg-mpm-cop-nul-nve-nve.txt > modules.spf
    humann_to_stamp.pl ./hmp_metagenomics/pre-computed_results/humann_output/01b-hit-keg-cat.txt > kos.spf

You should now have the 3 files 'pathways.spf', 'modules.spf', and 'kos.spf' in the current directory. You can check to make sure they are there with 'ls'

    ls

Your output should look like this:

```
ubuntu@ip-10-63-166-212:~/workspace/module_function$ ls
hmp_metagenomics  humann-0.99  kos.spf  modules.spf  pathways.spf  pre_humann
```

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