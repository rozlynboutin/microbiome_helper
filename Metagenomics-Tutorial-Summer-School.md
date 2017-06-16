**THIS PAGE IS CURRENTLY UNDER CONSTRUCTION **

# Introduction

This tutorial is set-up to walk you through the process of determining the taxonomic and functional composition of several metagenomic samples. It covers the use of [Metaphlan2](http://huttenhower.sph.harvard.edu/metaphlan2) (for taxonomic classification), [HUMAnN2](http://huttenhower.sph.harvard.edu/humann2) (for functional classification), and [STAMP](http://kiwi.cs.dal.ca/Software/STAMP) (for visualization and statistical evaluation).

Throughout the tutorial, there are several questions to ensure that you are understanding the process (and not just copy and pasting). You can check the [answer sheet](https://github.com/mlangill/microbiome_helper/wiki/Metagenomics-Summer-School-Tutorial-Answers) after you have answered them yourself.

We'll be using a subsampled version of the metagenomics dataset from [Schmidt et al. (PLoS ONE 2014)](http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0098741) that investigated changes in the oral microbiome associated with oral cancers.  

**Authors**: Morgan Langille and Gavin Douglas

**First Created**: Summer 2017

## Requirements

* Basic unix skills (This is a good introductory tutorial: http://korflab.ucdavis.edu/bootcamp.html)
* [Tutorial Data]()

## Initial Setup
Download the tutorial data, save it to the Desktop (within Ubuntu), unzip the folder, and enter this folder. You can do this using the below commands.  
  
Open a terminal/console and change to the directory containing the tutorial data:  
(paste the below command with your mouse, you probably wont be able to copy/paste text in the VBox with keyboard commands)  

```
cd ~/Desktop/  
wget XXXXX -O mgs_tutorial.zip  
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

**Q2)** How many samples are from each of the different sample sites?

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
@SRR3586062.7587400
CCAGCAAACACCTAATCTTACAGATAATTATGGTGCTTGGGAAGCGTTAGATAATTTTAAAAACGTATGGAAAATAGTTTATAGTTTGCTTTTTGTTGTTG
+
@@CFDEFFGGHHHGGEHJIIJGJGGIIJJJGIIB?FGIGJIFHIJJEHJJ<FGIGIIJJJJJJIJGIEHFHHEEECDCFE@ACEDFEDDCDD@CBBC@DD9
```

**Q3)** To save time each sample has been subsampled to a relatively low number of reads. How many sequences are there in each FASTQ? 

For your own data you may identify outlier samples with low read depth at this stage. It would also be advisable to explore the quality of your data with a tool like [FASTQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/). 

## Pre-processing  

The strength of shotgun metagenomics is also its weakness: any DNA in your samples will be sequenced with similar efficiencies. This is a problem when host microbiome samples are taken since it's possible to get substantial amounts of host DNA included in your samples. You should screen out these contaminant sequences before running taxonomic and functional classification. It's also a good idea to screen out PhiX sequences in your data: this virus is a common sequencing control since it has such a small genome.

Below we will screen out reads that map to the human and/or PhiX genomes. If your samples were taken from a different host you will need to map your reads to that genome instead.

```
echo "--very-sensitive-local" >> ./bowtie2_config.txt
run_contaminant_filter.pl -p 4 -o screened_reads/ subsampled_fastqs/* -d /home/shared/bowtiedb/GRCh38_PhiX -c ./bowtie2_config.txt 
```
The numbers and percentages of reads removed from each FASTQ are reported in the _screened\_reads.log_ file by default.

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
The counts and percentages of reads dropped is reported in _trimmomatic\_tabular\_log.txt_ by default.  

## Taxonomic Profiling with Metaphlan2
We will use Metaphlan2 to determine the taxonomic composition of each sample.  As with all other tools, Metaphlan2 has already been installed within the Microbiome Helper Virtual Box. 

Similar to other bioinformatic programs you can get the full help for the program by typing the ‘-h’ option after the program name:
 
    metaphlan2.py -h

As you can see there are quite a few options. We won’t explore all of these in this tutorial, but it may be worth reading on your own time. 

Find the option which tells you the version of Metaphlan2 you are using. 

**Q4)** What version of Metaphlan2 are you using?

### Running Metaphlan2

Now we are going to run Metaphlan2 on the sample "SRS015044". 

First change back to your working directory:

    cd ~/Desktop/hmp_metagenomics_downsampled

Now we are going to run Metaphlan2 on a single example with the following (long) command:
    
    metaphlan2.py --mpa_pkl /usr/local/prg/metaphlan2/db_v20/mpa_v20_m200.pkl --input_type fastq --bowtie2db /usr/local/prg/metaphlan2/db_v20/mpa_v20_m200 --no_map --nproc 2 -o SRS015044.txt fastq/SRS015044.fastq

The command will run for 1-2 minutes on this single sample (it'll be faster if you have more CPUs enabled, see the _nproc_ option described below).

The command line parameters are:
* `--mpa_pkl /usr/local/prg/metaphlan2/db_v20/mpa_v20_m200.pkl `: Indicates where the Metaphlan2 marker database is located which contains metadata information about each of the markers
* `--input_type fastq`: Indicates that our input files are in fastq format. (If we wanted to use fasta files as input then we would change this to 'fasta').
* `--bowtie2db /usr/local/prg/metaphlan2/db_v20/mpa_v20_m200`: This indicates the location of the marker database formatted for use with bowtie2
* `--no_map` We use this flag to prevent Metaphlan2 from storing the intermediate bowtie output files. 
* '--nproc 2' Indicates that 2 cores should be used to run this program, but will only work if you have enabled at least 2 cores already, otherwise only 1 core will be used.
* `-o SRS015044.txt`: Indicates the name of output file that Metaphlan2 will use to write results to.
* `fastq/SRS015044.fastq`: Metaphlan2 takes the input file containing our metagenomic reads as the last argument

Now run Metaphlan2 for sample SRS015893 as well (swap out the sample names for the above command).

### Metaphlan2 Output
You can inspect the output of these two samples by using the _less_ command (or your favourite editor): 

    less SRS015044.txt
    less SRS015893.txt

Your output should looks something like this:

    #SampleID       Metaphlan2_Analysis
    k__Bacteria     100.0
    k__Bacteria|p__Actinobacteria   33.07678
    k__Bacteria|p__Firmicutes       24.51694
    k__Bacteria|p__Proteobacteria   18.5776
    k__Bacteria|p__Bacteroidetes    11.65762
    k__Bacteria|p__Candidatus_Saccharibacteria      7.73074
    k__Bacteria|p__Fusobacteria     4.44031
    k__Bacteria|p__Actinobacteria|c__Actinobacteria 33.07678
    k__Bacteria|p__Firmicutes|c__Bacilli    13.6641
    k__Bacteria|p__Proteobacteria|c__Betaproteobacteria     12.55122
    k__Bacteria|p__Bacteroidetes|c__Flavobacteriia  11.23596

You can see that the output contains two columns, with the first column being the taxonomy name and the second column representing the relative abundance of that taxa (out of 100 total).

This output file provides summaries at each taxonomic level (e.g. phylum, class, family, genus, species, and sometimes strain level). At each taxonomic level the relative abundances will sum to 100. 

If reads can only be assigned to a certain taxonomic level (e.g. say the family level), then metaphlan will put those unassigned reads into a lower level taxonomic rank with the name "unclassified" appended.

For example lets pull out all the reads assigned to the genus _Veillonella_ using the _grep_ command:

    grep g__Veillonella SRS015044.txt

You should get output like this:

```
k__Bacteria|p__Firmicutes|c__Negativicutes|o__Selenomonadales|f__Veillonellaceae|g__Veillonella	10.85284
k__Bacteria|p__Firmicutes|c__Negativicutes|o__Selenomonadales|f__Veillonellaceae|g__Veillonella|s__Veillonella_parvula	9.97732
k__Bacteria|p__Firmicutes|c__Negativicutes|o__Selenomonadales|f__Veillonellaceae|g__Veillonella|s__Veillonella_unclassified	0.87552
k__Bacteria|p__Firmicutes|c__Negativicutes|o__Selenomonadales|f__Veillonellaceae|g__Veillonella|s__Veillonella_parvula|t__Veillonella_parvula_unclassified	9.97732
```

You can see that 10.85% of the metagenome is predicted from organisms in the genus _Veillonella_. Of those ~9.98% are assigned to the species _Veillonella parvula_, while ~0.88% are assigned to s__Veillonella_unclassified which means Metaphlan2 doesn't know what species to actually call them. 

**Q5)** What is the relative abundance of organisms unclassified at the species level for the genus _Neisseria_ in sample SRS015044? (Remember to use the grep command)
 
**Q6)** What phylum has the highest relative abundance in sample SRS015044? And SRS015893?

### Merging Metaphlan2 Results 

Run Metaphlan2 on sample SRS097871 as well, just like for the other 2 samples.

Now, lets combine all of the Metaphlan2 output files into a single merged output file:

    /usr/local/prg/metaphlan2/utils/merge_metaphlan_tables.py SRS015044.txt SRS015893.txt SRS097871.txt > metaphlan_merged.txt

Note that this script 'merge_metaphlan_tables.py' takes one or more Metaphlan2 output files as input and combines them into a single output file. The output file is indicated using the stdout redirect symbol '>' and is written in this case to **metaphlan_merged.txt**

The merged output file should look like this:
```
ID      SRS015044       SRS015893       SRS097871
#SampleID       Metaphlan2_Analysis     Metaphlan2_Analysis     Metaphlan2_Analysis
k__Bacteria     100.0   100.0   100.0
k__Bacteria|p__Actinobacteria   33.07678        7.41456 100.0
k__Bacteria|p__Actinobacteria|c__Actinobacteria 33.07678        7.41456 100.0
k__Bacteria|p__Actinobacteria|c__Actinobacteria|o__Actinomycetales      33.07678        7.41456 100.0
k__Bacteria|p__Actinobacteria|c__Actinobacteria|o__Actinomycetales|f__Actinomycetaceae  1.75917 7.41456 2.12537
k__Bacteria|p__Actinobacteria|c__Actinobacteria|o__Actinomycetales|f__Actinomycetaceae|g__Actinomyces   1.75917 7.41456 2.12537
k__Bacteria|p__Actinobacteria|c__Actinobacteria|o__Actinomycetales|f__Actinomycetaceae|g__Actinomyces|s__Actinomyces_graevenitzii
       0.0     7.41456 0.0
```
Now each sample is listed as a different column within this output file. You can view this file again using 'less' or you can import it into your favourite spreadsheet program.

**Q7)** What phylum is present in one of the samples but is absent in the others? What sample is this phylum in? 

Reminder: to get information about the individual samples you can look in the hmp_map.txt file using 'less':

    less hmp_map.txt

**Q8)** What body site is the sample taken from that contains the unique phylum from question 7?

### Running Metaphlan2 on a large number of samples using Microbiome Helper

Running Metaphlan2 on more than a few samples can be tedious. I wrote a script to make this easier as part of the package “Microbiome Helper” (https://github.com/mlangill/microbiome_helper). **Note you do not have to do this today** as we have already pre-computed the combined Metaphlan2 output, but this is how you would do it.

To run Metaphlan2 on all of the samples using 'run_metaphlan2.pl' (this would take ~30 min):

    run_metaphlan2.pl -p 2 -o metaphlan_merged_all.txt ./fastq/*

This command uses the following options:
* '-p 2' runs metaphlan in parallel using 2 processors.
* '-o' is the name for the merged output file.
* ' ./fastq/*' indicates that all of the files within this directory will be used as input. 

On your screen you would see the commands that the microbiome helper script is running automatically for you:
```
 cat ./fastq/SRS014477.fastq | /usr/local/metaphlan2/metaphlan2.py  --input_type multifastq --mpa_pkl /usr/local/metaphlan2/db_v20/mpa_v20_m200.pkl --bt2_ps sensitive-local --min_alignment_len 50 --bowtie2db /usr/local/metaphlan2/db_v20/mpa_v20_m200 --no_map > ./metaphlan_out/SRS014477
 cat ./fastq/SRS011343.fastq | /usr/local/metaphlan2/metaphlan2.py  --input_type multifastq --mpa_pkl /usr/local/metaphlan2/db_v20/mpa_v20_m200.pkl --bt2_ps sensitive-local --min_alignment_len 50 --bowtie2db /usr/local/metaphlan2/db_v20/mpa_v20_m200 --no_map > ./metaphlan_out/SRS011343
 cat ./fastq/SRS019129.fastq | /usr/local/metaphlan2/metaphlan2.py  --input_type multifastq --mpa_pkl /usr/local/metaphlan2/db_v20/mpa_v20_m200.pkl --bt2_ps sensitive-local --min_alignment_len 50 --bowtie2db /usr/local/metaphlan2/db_v20/mpa_v20_m200 --no_map > ./metaphlan_out/SRS019129
```
**Q9)** Based on the above example commands being written to the screen, what directory would the Metaphlan2 individual output files be stored in?

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

## Determining Functional Composition with HUMAnN

We will now functionally annotate the metagenomes using HUMAnN. HUMAnN analyzes metagenomic data in the context of biochemical pathways, elucidating the distribution of roles among microbial community members. 

If you are running this tutorial on our virtual box with low RAM and a low number of CPUs then you should close everything else except for Terminal. Also, if you don't have at least 2GB of memory available for the virtual box (you can change this before running the virtual box, but of course this is limited by the specs of your actual computer) the following commands could be quite slow (especially the "scons" command). If you notice your commands seem to be frozen you can quit them with "CTRL-C" and skip that step, you will be able to analyze the data with STAMP in either case.  

### Running DIAMOND search against KEGG

If you are short on time you can skip to [interpreting the functional results](#running-all-samples-with-microbiome-helper). 

Before running HUMAnN, it is first necessary run a similarity search against the KEGG database. This database is a collection of pathways for metabolic and other functional molecular and cellular processes, and is located at “/home/shared/kegg/diamond_db” on the virtual box. This is a reduced KEGG database that the authors of HUMAnN created and which they make available through personal correspondence. This similarity search can be done using various tools such as BLAST, USEARCH, RapSearch, etc. In this tutorial we will use a relatively new similarity search tool that is much faster and works well for large metagenomic datasets called [DIAMOND](http://ab.inf.uni-tuebingen.de/software/diamond/).

First we will make a directory to store our sequence search outputs:

    mkdir pre_humann

Now we will run DIAMOND on our metagenomic sequences for the sample SRS015044:  
  
    diamond blastx -p 2 -d /home/shared/kegg/diamond_db/kegg.reduced -q ./fastq/SRS015044.fastq -o pre_humann/SRS015044.txt -b 0.4  
   
The options used in this command are:

* 'blastx': Tells DIAMOND to run in “blastx” mode meaning that we will search a nucleotide query against a protein database in all 6 frame translations (3 forward and 3 reverse).
* '-p 2' indicates that DIAMOND should use 2 threads to do the search.
*  '-d kegg/kegg.reduced' points at the KEGG database which has already been formatted for use with DIAMOND. 
* '-q ./fastq/SRS015044.fastq' is the input metagenomic sample.
* '-o pre_humann/SRS015044.txt' is the name of the output file.  
* '-b 0.4' indicates that the sequence block-size in billions of letters should be 5 times lower than the default, which will mean the program will run slower, but use less memory.   
* Note: The default e-value cutoff used by DIAMOND is 0.001.  
  
A lot of information is printed to screen while this command is running. You can silence this information by using the "--quiet" option.  
    
The previous command is comparing 60k sequences vs. the KEGG database (~1.3 million sequences) and takes about 10 minutes with 2 CPUs. If we had used the NCBI BLASTX it would have taken a couple of days!

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

Now run DIAMOND for samples SRS015893 and SRS097871 against the KEGG database as above.   
  
### Running HUMAnN
  
Now we are going to run HUMAnN on the three output files. To use HUMAnN you are going to make a symbolic link between the BLAST tabular format files and the HUMAnN "input" directory (HUMAnN reads in input files from this directory).  
  
    rm /home/shared/humann-0.99/input/*txt 
    rm /home/shared/humann-0.99/output/*
    ln -s $PWD/pre_humann/*txt /home/shared/humann-0.99/input
  
(It's ok if the 'rm' commands give the error "No such file or directory", I just added that in to make sure no previous files will mess up HUMAnN).

Then move into the HUMAnN directory:
 
    cd /home/shared/humann-0.99

To being running HUMAnN on all the samples you pre-processed with DIAMOND, you use the “scons” command. Note this will take ~20 minutes.
  
    scons  
  
A bunch of messages will pass on your screen as this command runs. All of the output is contained in the “/home/shared/humann-0.99/output” directory. To see a list of them, once the job is done, type:
 
    ls output

There are MANY output files from HUMAnN. The ones we care about are called:

* **01b-hit-keg-cat.txt** –> KEGG KOs
* **04b-hit-keg-mpm-cop-nul-nve-nve.txt** -> KEGG Modules
* **04b-hit-keg-mpt-cop-nul-nve-nve.txt** -> KEGG Pathways

These files contain relative abundances for each of these different functional classifications. You can look at the format of these using _less_. For example to look at the KEGG Pathways:

    less output/04b-hit-keg-mpt-cop-nul-nve-nve.txt

The output should look like this:
    
    ID      NAME    SRS015044-hit-keg-mpt-cop-nul-nve-nve   SRS015893-hit-keg-mpt-cop-nul-nve-nve   SRS097871-hit-keg-mpt-cop-nul-nve-nve  
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
    InverseSimpson  InverseSimpson  63.1705 59.9415 52.4014  
    Shannon Shannon 4.29471 4.24922 4.15983  
    Pielou  Pielou  0.878201        0.8689  0.85062  
    Richness        Richness        1       1       1  
    ko00564 ko00564: Glycerophospholipid metabolism 0.00852428      0.00633934      0.00662779  
    ko00680 ko00680: Methane metabolism     0.00515712      0.0052314       0.00512276  
    ko00562 ko00562: Inositol phosphate metabolism  0.00293604      0.00345021      0.0054698  
    ko00563 ko00563: Glycosylphosphatidylinositol(GPI)-anchor biosynthesis  0.000189354     0       0  
    ko00561 ko00561: Glycerolipid metabolism        0.00498252      0.00491085      0.00503682  
    ko00440 ko00440: Phosphonate and phosphinate metabolism 0.00311794      0.00103415      0.00611991  
    ko00250 ko00250: Alanine, aspartate and glutamate metabolism    0.020349        0.0223569       0.0203559  
    ko00565 ko00565: Ether lipid metabolism 0.000426284     0.000316276     0  
    ko00072 ko00072: Synthesis and degradation of ketone bodies     0.00366787      0       0  
    ko04111 ko04111: Cell cycle - yeast     0       4.32745e-06     0  
    ko00010 ko00010: Glycolysis / Gluconeogenesis   0.016986        0.0146625       0.0163483  
    

* The first line indicates that the first column is a short id for the KEGG Pathway, the second column is a longer description of the KEGG Pathway, and the remaining columns represent the sample identifiers. 
* The next 13 lines contain metadata for each sample.
* The following 4 lines (starting at "InverseSimpson") calculate different alpha-diversity metrics for each sample, but in general these are usually not that useful and can be ignored.
* From line 19 onward, each row indicates the different relative abundance for each KEGG Pathway. Note that these values are small because they have been normalized so that each sample will sum to 1. 
* It should be noted that whether metadata rows are included or not depends on your settings.

Now, use less to look at the KO predictions and the KEGG Module predictions:

    less output/01b-hit-keg-cat.txt
    less output/04b-hit-keg-mpm-cop-nul-nve-nve.txt

**Q1)** Which of the three samples has the lowest relative abundance of the KEGG Module: “M00319: Manganese/zinc/iron transport system”?

### Running all samples with Microbiome Helper

So what about all of our samples? To do the DIAMOND searches for all 30 you could use a wrapper script provided by the microbiome_helper package called run_pre_humann.pl. All 30 samples _could be_ processed with a single command like:
  
     ls fastq/*fastq | parallel --eta -j 2 --load 90% diamond blastx -p 1 -d /home/shared/kegg/diamond_db/kegg.reduced -b 0.4 -q {} -o pre_humann/{/.}.txt --quiet  
  
However, this would take several hours to complete (this is much faster when more CPUs are used). The HUMAnN step would take at least 10 minutes to complete.  
  
To make things easier the output for all 30 samples has been pre-computed and is located in  “./pre-computed_results/humann_output”. 

If you browse the output using _less_ you can see that they are in the same format but with 30 columns representing the 30 samples (below "~" specifies your home directory):

    cd ~/Desktop/hmp_metagenomics_downsampled
    less ./pre-computed_results/humann_output/04b-hit-keg-mpt-cop-nul-nve-nve.txt

You will notice that the output looks fairly messy because the terminal will automatically line wrap and it becomes hard to see where one line ends and the next begins. I often find the "cut" command useful to browse data like this. "cut" allows you to just look at particular columns from the data. For example:

    cut -f 1-5 ./pre-computed_results/humann_output/04b-hit-keg-mpt-cop-nul-nve-nve.txt | less

* '-f 1-5' indicates that we want to look at the first 5 columns. We could just have easily used '-f 1,3,10' (to view columns 1, 3 and 10) or '-f 1-3,20-' (to view columns 1 to 3 and columns 20 onwards).
* '| less' "pipes" the output from the "cut" command and lets us view the output one screen at a time using our 'less' tool.  

Now, we are going to use STAMP to visualize the humann output just like we did with the taxonomic data. 

First, we need to convert the HUMAnN output files file into STAMP format:

    humann_to_stamp.pl ./pre-computed_results/humann_output/04b-hit-keg-mpt-cop-nul-nve-nve.txt > pathways.spf
    humann_to_stamp.pl ./pre-computed_results/humann_output/04b-hit-keg-mpm-cop-nul-nve-nve.txt > modules.spf
    humann_to_stamp.pl ./pre-computed_results/humann_output/01b-hit-keg-cat.txt > kos.spf

You should now have the 3 files 'pathways.spf', 'modules.spf', and 'kos.spf' in the current directory. You can check to make sure they are there with 'ls'

    ls

Your output should look like this:

    mh_user@MicrobiomeHelper:~/Desktop/hmp_metagenomics_downsampled$ ls
    fastq        kos.spf                   metaphlan_merged.txt  pathways.spf          pre_humann     SRS015893.txt
    hmp_map.txt  metaphlan_merged_all.spf  modules.spf           pre-computed_results  SRS015044.txt  SRS097871.txt



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