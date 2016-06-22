  * [Introduction](#introduction)
    * [Requirements](#requirements)
    * [Initial Setup](#initial-setup)
    * [Explore Samples](#explore-samples)
  * [Taxonomic Profiling with Metaphlan2](#taxonomic-profiling-with-metaphlan2)
    * [Running Metaphlan2](#running-metaphlan2)
    * [Metaphlan2 Output](#metaphlan2-output)
    * [Merging Metaphlan2 Results](#merging-metaphlan2-results)
    * [Running Metaphlan2 on a large number of samples](#running-metaphlan2-on-a-large-number-of-samples-using-microbiome-helper)
    * [Analyzing Metaphlan2 output with STAMP](#analyzing-metaphlan2-output-with-stamp)

# Introduction

This tutorial is set-up to walk you through the process of determining the taxonomic and functional composition of several metagenomic samples. It covers the use of [Metaphlan2](http://huttenhower.sph.harvard.edu/metaphlan2), and [STAMP](http://kiwi.cs.dal.ca/Software/STAMP) (for visualization and statistical evaluation).  

Throughout the tutorial, there are several questions to ensure that you are understanding the process (and not just copy and pasting). You can check the [answer sheet](Metagenomics-Tutorial-(Downsampled)-answers) after you have answered them yourself.

This lab component will use samples collected and sequenced through the Human Microbiome Project (HMP).  We will be analyzing samples collected from three oral sites including subgingival, supragingival, and the tongue. These samples have been downloaded for you already from the HMP-DACC (http://hmpdacc.org/HMIWGS/all/) and have been randomly subsampled to a much smaller number of reads to allow faster computation time. Real data will require more waiting time between each step!

**Author**: Morgan Langille

**Contributions by**: Gavin Douglas

**First Created**: Summer 2015 for CBW Metagenomics, Halifax

**Last Edited**: Summer 2016 for CBW Metagenomics, Vancouver

## Requirements

* Basic unix skills (This is a good introductory tutorial: http://korflab.ucdavis.edu/bootcamp.html)

* These samples have been downloaded for you already from the HMP-DACC (http://hmpdacc.org/HMIWGS/all/) and placed in the directory: ~/CourseData/metagenomics/hmp_metagenomics/

## Initial Setup

* Prepare working directory

    rm -rf ~/workspace/module_taxonomy
    mkdir -p ~/workspace/module_taxonomy
    cd ~/workspace/module_taxonomy
    ln -s ~/CourseData/metagenomics/hmp_metagenomics ./

'''Notes''':

* The `ln -s` command adds a symbolic link to the (read-only) `~/CourseData/metagenomics/hmp_metagenomics` directory.

### Explore Samples

Change to the directory containing the tutorial data:

(paste the below command with your mouse, you probably wont be able to copy/paste text in the VBox with keyboard commands)

    cd ~/workspace/module_taxonomy/hmp_metagenomics

The tutorial data consists of a "map file" containing information about each of the samples called **hmp_map.txt**, along with the sequence data for each sample being contained within a separate fastq file within the directory **fastq**.

Lets take a look at the **hmp_map.txt** using _less_ (type q to quit out of the program when done)

    less hmp_map.txt

You will have noticed that this map file contains three columns with the first being sample ids, the middle column being the body site, and the third being the sex of the person. 

Remember you can count the lines of a file using _wc -l_. For example,

    wc -l hmp_map.txt

**Q1)** How many samples are there in total (you can either look at the hmp_map.txt file or count the fastq files)?

**Q2)** How many samples are from each of the different sample sites?

**Q3)** How many sequences are there in each of the samples? Remember there are 4 lines in a fastq file for each sequence. 

## Taxonomic Profiling with Metaphlan2
We will use Metaphlan2 to determine the taxonomic composition of each sample.  As with all other tools, Metaphlan2 has already been installed within the Microbiome Helper Virtualbox. 

Similar to other bioinformatic programs you can get the full help for the program by typing the ‘-h’ option after the program name:
 
    metaphlan2.py -h

As you can see there are quite a few options. We won’t explore all of these in this tutorial, but it may be worth reading on your own time. 

Find the option which tells you the version of Metaphlan2 you are using. 

**Q4)** What version of Metaphlan2 are you using?

### Running Metaphlan2

Now we are going to run Metaphlan2 on the sample "SRS015044". 

First change back to your working directory:

    cd ~/workspace/module_taxonomy

Now we are going to run Metaphlan2 on a single example with the following (long) command:
    
    metaphlan2.py --mpa_pkl /usr/local/metaphlan2/db_v20/mpa_v20_m200.pkl --input_type fastq --bowtie2db /usr/local/metaphlan2/db_v20/mpa_v20_m200 --no_map -o SRS015044.txt hmp_metagenomics/fastq/SRS015044.fastq

The command will run for 1-2 minutes on this single sample (it'll be faster if you have more CPUs enabled, see the _nproc_ option described below).

The command line parameters are:
* `--mpa_pkl /usr/local/metaphlan2/db_v20/mpa_v20_m200.pkl `: Indicates where the Metaphlan2 marker database is located which contains metadata information about each of the markers
* `--input_type fastq`: Indicates that our input files are in fastq format. (If we wanted to use fasta files as input then we would change this to 'fasta').
* `--bowtie2db /usr/local/metaphlan2/db_v20/mpa_v20_m200`: This indicates the location of the marker database formatted for use with bowtie2
* `--no_map` We use this flag to prevent Metaphlan2 from storing the intermediate bowtie output files. 
* '--nproc 2' Indicates that 2 cores should be used to run this program, but will only work if you have enabled at least 2 cores already, otherwise only 1 core will be used.
* `-o SRS015044.txt`: Indicates the name of output file that Metaphlan2 will use to write results to.
* `fastq/SRS015044.fastq`: Metaphlan2 takes the input file containing our metagenomic reads as the last argument

Now run Metaphlan2 for sample SRS015893 as well (swap out the sample names for the above command). Also this time lets use all 8 cores of our server by adding the option '-–nproc 8'.

     metaphlan2.py --mpa_pkl /usr/local/metaphlan2/db_v20/mpa_v20_m200.pkl --input_type fastq --bowtie2db /usr/local/metaphlan2/db_v20/mpa_v20_m200 --no_map -o SRS015893.txt --nproc 8 hmp_metagenomics/fastq/SRS015893.fastq

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

Run Metaphlan2 on sample **SRS097871** as well, just like for the other 2 samples we have already done. **(You need to figure out the command and run it yourself before moving to the next step)**

Now, lets combine all of the Metaphlan2 output files into a single merged output file:

    /usr/local//metaphlan2/utils/merge_metaphlan_tables.py SRS015044.txt SRS015893.txt SRS097871.txt > metaphlan_merged.txt

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

To get information about the individual samples you can look in the hmp_map.txt file using 'less':

 less hmp_map.txt

**Q8)** What body site is the sample taken from that contains the unique phylum from question 7?

### Running Metaphlan2 on a large number of samples using Microbiome Helper

Running Metaphlan2 on more than a few samples can be tedious. There is a script within Microbiome Helper to help automate and simplify this.

To run Metaphlan2 on all of the samples using 'run_metaphlan2.pl' (this would take ~30 min):

    run_metaphlan2.pl -p 8 -o metaphlan_merged_all.txt ./hmp_metagenomics/fastq/*

This command uses the following options:
* '-p 8' runs metaphlan in parallel using 8 processors.
* '-o' is the name for the merged output file.
* ' ./hmp_metagenomics/fastq/*' indicates that all of the files within this directory will be used as input. 

On your screen you would see the commands that the microbiome helper script is running automatically for you:
```
cat ./hmp_metagenomics/fastq/SRS015537.fastq | /usr/local/metaphlan2/metaphlan2.py  --input_type multifastq --mpa_pkl /usr/local/metaphlan2/db_v20/mpa_v20_m200.pkl --bt2_ps sensitive-local --min_alignment_len 50 --bowtie2db /usr/local/metaphlan2/db_v20/mpa_v20_m200 --no_map > ./metaphlan_out/SRS015537
cat ./hmp_metagenomics/fastq/SRS014477.fastq | /usr/local/metaphlan2/metaphlan2.py  --input_type multifastq --mpa_pkl /usr/local/metaphlan2/db_v20/mpa_v20_m200.pkl --bt2_ps sensitive-local --min_alignment_len 50 --bowtie2db /usr/local/metaphlan2/db_v20/mpa_v20_m200 --no_map > ./metaphlan_out/SRS014477
cat ./hmp_metagenomics/fastq/SRS015064.fastq | /usr/local/metaphlan2/metaphlan2.py  --input_type multifastq --mpa_pkl /usr/local/metaphlan2/db_v20/mpa_v20_m200.pkl --bt2_ps sensitive-local --min_alignment_len 50 --bowtie2db /usr/local/metaphlan2/db_v20/mpa_v20_m200 --no_map > ./metaphlan_out/SRS015064
(etc.)
```
**Q9)** Based on the above example commands being written to the screen, what directory would the Metaphlan2 individual output files be stored in?

### Analyzing Metaphlan2 output with STAMP

STAMP takes two main files as input the profile data which is a table that contains the abundance of features (i.e. taxonomic or functions) and a group metadata file which provides more information about each of the samples in the profile data file. 

The metadata file is the **hmp_map.txt** file. This file is present already in the "hmp_metagenomics_downsampled" directory.
You will also need the profile data file which is in a different format than what Metaphlan2 outputs. Therefore we first need to convert it using a script from the microbiome_helper package. You can convert the pre-computed Metaphlan2 output for all samples like this:

    metaphlan_to_stamp.pl metaphlan_merged_all.txt > metaphlan_merged_all.spf

As you can see this file takes the Metaphlan2 output file (from all the merged samples), as the only parameter and then writes the output to a new file called "metaphlan_merged_all.spf".

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

