
## Introduction

This tutorial is set-up to walk you through the process of determining the taxonomic and functional composition of several metagenomic samples. It covers the use of [Metaphlan2](http://huttenhower.sph.harvard.edu/metaphlan2) (for taxonomy) and [Humann](https://huttenhower.sph.harvard.edu/humann) (for functional) and [STAMP](http://kiwi.cs.dal.ca/Software/STAMP) (for visualization and statistical evaluation).  

Throughout the tutorial, there are several questions to ensure that you are understanding the process (and not just copy and pasting). You can check the [answer sheet](Metagenomics_Taxonomic_Composition_Lab_Answers) after you have answered them yourself.

This lab component will use samples collected and sequenced through the Human Microbiome Project.  We will be analyzing samples collected from three oral sites including subgingival, supragingival, and the tongue. These samples have been downloaded for you already from the HMP-DACC (http://hmpdacc.org/HMIWGS/all/) and have been randomly subsampled to a much smaller number of reads to allow faster computation time. Real data will require more waiting time between each step!

## Requirements

* Basic unix skills (This is a good introductory tutorial: http://korflab.ucdavis.edu/bootcamp.html)
* [Microbiome Helper VirtualBox] (MicrobiomeHelper-Virtual-Box) (install it and ensure it is working)
* [Tutorial Data](https://www.dropbox.com/s/xztirse75q7k3bn/hmp_metagenomics.zip?dl=1)

## Initial Setup

* If you haven't already, launch the Microbiome Helper VirtualBox and open this tutorial within Ubuntu
* Download the tutorial data and place it in: 

Create a new directory that will store all of the files created in this lab.

    rm -rf ~/workspace/module3
    mkdir -p ~/workspace/module3
    cd ~/workspace/module3
    ln -s ~/CourseData/hmp_metagenomics ./

## Main Lab Steps
### Explore Samples

The “hmp_metagenomics/fastq” directory contains all of the fastq files (one for each sample) and the hmp_map.txt which describes the metadata for each sample.

Change to this directory so we can explore the files in more details:

    cd hmp_metagenomics

Using your unix skills attempt to answer the following questions.

Remember you can view the contents of a file using 'less' (type q to quit out of the program). For example,

    less hmp_map.txt

Note 2, you can count the lines of a file using 'wc -l'. For example,

    wc -l hmp_map.txt

Q1) How many samples are there in total (you can either look at the hmp_map.txt file or count the fastq files)?

<div class="toccolours mw-collapsible mw-collapsed" style="width:200px">
Hint:
<div class="mw-collapsible-content">
* Can manually count lines in the hmp_map.txt file or count the number of files in the fastq directory
OR
* Can ‘wc –l’ the hmp_map.txt file and subtract 1 for the header 
OR
* Can ‘ls fastq/* | wc –l’
</div>
</div>

Q2) How many sample are from each of the different sample sites?
<div class="toccolours mw-collapsible mw-collapsed" style="width:200px">
Hint:
<div class="mw-collapsible-content">
    less hmp_map.txt
</div>
</div>
Q3) How many sequences are there in each of the samples?
<div class="toccolours mw-collapsible mw-collapsed" style="width:200px">
Hint:
<div class="mw-collapsible-content">
Remember there are 4 lines in a fastq file for each sequence. 
</div>
</div>

### Taxonomic Profiling with Metaphlan2
We will use Metaphlan2 to determine the taxonomic composition of each sample.  Metaphlan2 has already been installed. Similar to other bioinformatic programs you can get the full help for the program by typing the ‘-h’ option after the program name:
 metaphlan2.py –h

As you can see there are quite a few options. We won’t explore all of these in this tutorial, but it may be worth reading on your own time. 

Find the option which tells you the version of Metaphlan2 you are using. 

Q4) From the help what is the exact version of metaphlan that you are using and when was it released?

### Running Metaphlan

Now we are going to run metaphlan on the sample '''SRS015044'''. 

First change back to your module 3 directory:

    cd ~/workspace/module3/

Now run metaphlan with the following (long) command:
    
    metaphlan2.py --mpa_pkl /usr/local/metaphlan2/db_v20/mpa_v20_m200.pkl --input_type fastq --bowtie2db /usr/local/metaphlan2/db_v20/mpa_v20_m200 --no_map -o SRS015044.txt hmp_metagenomics/fastq/SRS015044.fastq

The command will run for 1-2 minutes on this single sample.

The command line parameters are:
* `--mpa_pkl /usr/local/metaphlan2/db_v20/mpa_v20_m200.pkl `: Indicates where the Metaphlan marker database is located which contains metadata information about each of the markers
* `--input_type fastq`: Indicates that our input files are in fastq format. (If we wanted to use fasta files as input then we would change this to 'fasta').
* `--bowtie2db /usr/local/metaphlan2/db_v20/mpa_v20_m200`: This indicates the location of the marker database formatted for use with bowtie2
* `--no_map` We use this flag to prevent metaphlan from storing the intermediate bowtie output files. 
* `-o SRS015044.txt`: Indicates the name of output file that metaphlan will use to write results to.
* `hmp_metagenomics/fastq/SRS015044.fastq`: Metaphlan takes the input file containing our metagenomic reads as the last argument


Now we are going to run metaphlan on another sample, '''SRS015893''', but this time use all 4 cores of our server by adding the option ''''-–nproc 4'''':
    metaphlan2.py --mpa_pkl /usr/local/metaphlan2/db_v20/mpa_v20_m200.pkl --input_type fastq --bowtie2db /usr/local/metaphlan2/db_v20/mpa_v20_m200 --no_map -o SRS015893.txt --nproc 4 hmp_metagenomics/fastq/SRS015893.fastq  

### Metaphlan Output
You can inspect the output of these two commands by using the ''less'' command (or your favourite editor): 

    less SRS015044.txt
    less SRS015893.txt

Your output should looks something like this:

    #SampleID       Metaphlan2_Analysis
    k__Bacteria     100.0
    k__Bacteria|p__Actinobacteria   33.58989
    k__Bacteria|p__Firmicutes       22.70338
    k__Bacteria|p__Proteobacteria   17.56533
    k__Bacteria|p__Bacteroidetes    11.92585
    k__Bacteria|p__Candidatus_Saccharibacteria      9.31222
    k__Bacteria|p__Fusobacteria     4.90334
    k__Bacteria|p__Actinobacteria|c__Actinobacteria 33.58989
    k__Bacteria|p__Firmicutes|c__Bacilli    12.74551
    k__Bacteria|p__Bacteroidetes|c__Flavobacteriia  10.80566
    k__Bacteria|p__Firmicutes|c__Negativicutes      9.95786


You can see that the output contains two columns, with the first column being the taxonomy name and the second column representing the relative abundance of that taxa (out of 100 total).

This output file provides summaries at each taxonomic level (e.g. phylum, class, family, genus, species, and sometimes strain level). At each taxonomic level the relative abundances will sum to 100. 

If reads can only be assigned to a certain taxonomic level (e.g. say the family level), then metaphlan will put those unassigned reads into a lower level taxonomic rank with the name "unclassified" appended.

For example lets pull out all the reads assigned to the family "Leptotrichiaceae" using the ''grep'' command:

    grep f__Leptotrichiaceae SRS015044.txt

You should get output like this:

 k__Bacteria|p__Fusobacteria|c__Fusobacteriia|o__Fusobacteriales|f__Leptotrichiaceae	3.28846
 k__Bacteria|p__Fusobacteria|c__Fusobacteriia|o__Fusobacteriales|f__Leptotrichiaceae|g__Leptotrichia	2.58466
 k__Bacteria|p__Fusobacteria|c__Fusobacteriia|o__Fusobacteriales|f__Leptotrichiaceae|g__Leptotrichiaceae_unclassified	0.7038
 k__Bacteria|p__Fusobacteria|c__Fusobacteriia|o__Fusobacteriales|f__Leptotrichiaceae|g__Leptotrichia|s__Leptotrichia_unclassified	1.99034
 k__Bacteria|p__Fusobacteria|c__Fusobacteriia|o__Fusobacteriales|f__Leptotrichiaceae|g__Leptotrichia|s__Leptotrichia_buccalis	0.49568
 k__Bacteria|p__Fusobacteria|c__Fusobacteriia|o__Fusobacteriales|f__Leptotrichiaceae|g__Leptotrichia|s__Leptotrichia_wadei	0.05468
 k__Bacteria|p__Fusobacteria|c__Fusobacteriia|o__Fusobacteriales|f__Leptotrichiaceae|g__Leptotrichia|s__Leptotrichia_hofstadii	0.04395
 k__Bacteria|p__Fusobacteria|c__Fusobacteriia|o__Fusobacteriales|f__Leptotrichiaceae|g__Leptotrichia|s__Leptotrichia_buccalis|t__GCF_000023905	0.49568
 k__Bacteria|p__Fusobacteria|c__Fusobacteriia|o__Fusobacteriales|f__Leptotrichiaceae|g__Leptotrichia|s__Leptotrichia_wadei|t__Leptotrichia_wadei_unclassified	0.05468
 k__Bacteria|p__Fusobacteria|c__Fusobacteriia|o__Fusobacteriales|f__Leptotrichiaceae|g__Leptotrichia|s__Leptotrichia_hofstadii|t__GCF_000162955	0.04395

You can see that 3.28846% of the metagenome is predicted from organisms in the family Leptotrichiaceae. Of those ~2.6% are assigned to the genus ''Leptotrichia'', while ~0.7% are assigned to ''g__Leptotrichiaceae_unclassifie'' which means metaphlan doesn't know what genus to actually call them. 

Q5) What is the relative abundance of organisms unclassified at the species level for the genus ''Neisseria'' in sample SRS015044? (Remember to use the grep command)
 
Q6) What phylum has the highest relative abundance in sample SRS015044? And SRS015893?

Now try running metaphlan using the same options as above but for sample SRS097871. (Make sure to change the input and output files)

### Merging Metaphlan Results 

Now lets combine all of the metaphlan output files into a single merged output file:

 /usr/local/metaphlan2/utils/merge_metaphlan_tables.py SRS015044.txt SRS015893.txt SRS097871.txt > metaphlan_merged.txt

Note that this script 'merge_metaphlan_tables.py' takes one or more metaphlan output files as input and combines them into a single output file. The output file is indicated using the stdout redirect symbol '>' and is written in this case to ''metaphlan_merged.txt''

The merged output file should look like this:

 ID      SRS015044       SRS015893       SRS097871
 #SampleID       Metaphlan2_Analysis     Metaphlan2_Analysis     Metaphlan2_Analysis
 k__Bacteria     100.0   100.0   100.0
 k__Bacteria|p__Actinobacteria   33.58989        23.49742        66.06867
 k__Bacteria|p__Actinobacteria|c__Actinobacteria 33.58989        23.49742        66.06867
 k__Bacteria|p__Actinobacteria|c__Actinobacteria|o__Actinomycetales      33.58989        22.83325        66.06867
 k__Bacteria|p__Actinobacteria|c__Actinobacteria|o__Actinomycetales|f__Actinomycetaceae  4.60715 22.27157        20.84159

Now each sample is listed as a different column within this output file. You can view this file again using 'less' or you can import it into your favourite spreadsheet program.

Q7) What phylum is present in one of the samples but is absent in the others? What sample is this phylum in? 

To get information about the individual samples you can look in the hmp_map.txt file using 'less':

 less hmp_metagenomics/hmp_map.txt

Q8) What body site is the sample taken from that contains the unique phylum from question 6?

====Running Metaphlan on a large number of samples using Microbiome Helper====

Running metaphlan on more than a few samples can be tedious. 

To make this process simpler we are going to use a script I wrote as part of the package “Microbiome Helper” (https://github.com/mlangill/microbiome_helper).

Run metahplan on all of the sample using 'run_metaphlan2.pl' (this will take about 15 minutes):

    run_metaphlan2.pl -p 8 -o metaphlan_merged_all.txt hmp_metagenomics/fastq/*

This command uses the following options:
* '-p 8' runs metaphlan in parallel using 8 processors.
* '-o' is the name for the merged output file.
* ' hmp_metagenomics/fastq/*' indicates that all of the files within this directory will be used as input. 

On your screen you should see the commands that the microbiome helper script is running automatically for you:

 cat hmp_metagenomics/fastq/SRS014477.fastq | /usr/local/metaphlan2/metaphlan2.py  --input_type multifastq --mpa_pkl /usr/local/metaphlan2/db_v20/mpa_v20_m200.pkl --bt2_ps sensitive-local --min_alignment_len 50 --bowtie2db /usr/local/metaphlan2/db_v20/mpa_v20_m200 --no_map > ./metaphlan_out/SRS014477
 cat hmp_metagenomics/fastq/SRS011343.fastq | /usr/local/metaphlan2/metaphlan2.py  --input_type multifastq --mpa_pkl /usr/local/metaphlan2/db_v20/mpa_v20_m200.pkl --bt2_ps sensitive-local --min_alignment_len 50 --bowtie2db /usr/local/metaphlan2/db_v20/mpa_v20_m200 --no_map > ./metaphlan_out/SRS011343
 cat hmp_metagenomics/fastq/SRS019129.fastq | /usr/local/metaphlan2/metaphlan2.py  --input_type multifastq --mpa_pkl /usr/local/metaphlan2/db_v20/mpa_v20_m200.pkl --bt2_ps sensitive-local --min_alignment_len 50 --bowtie2db /usr/local/metaphlan2/db_v20/mpa_v20_m200 --no_map > ./metaphlan_out/SRS019129

Q9) Based on the commands being written to the screen, what directory are the metaphlan individual output files being stored in?

===STAMP===
While Metaphlan is running lets prepare to use STAMP. You should have already installed STAMP on your laptop before the workshop, if not, then do so now.

STAMP takes two main files as input the profile data which is a table that contains the abundance of features (i.e. taxonomic or functions) and a group metadata file which provides more information about each of the samples in the profile data file. 

The metadata file is the ''''hmp_metagenomics/hmp_map.txt'''' file. Download this file locally to your computer using an appropriate method (e.g. WinSCP, etc.)
You will also need the profile data file which is in a different format than metaphlan outputs. Therefore we first need to convert it using a script from the microbiome helper package. Once metaphlan is finished running do the conversion like this:

    metaphlan_to_stamp.pl metaphlan_merged_all.txt > metaphlan_merged_all.spf

As you can see this file takes the metaphlan output file (from all the merged samples) as the only parameter and then writes the output to a new file called "metaphlan_merged_all.spf".

Now download the '''metaphlan_merged_all.spf''' file to your computer as well.

Now load the profile file (metaphlan_merged_all.spf" and the metadata file (hmp_metadata.txt) files by going to ”File->load data” within STAMP:

[[File:STAMP_File_Menu.png]]

Change the “Profile level” (top left) to “Genus”, ensure that the Group legend (top right) has been set to “sex”, and that “PCA plot” has been set below the large middle window. 

You should now be looking at a PCA plot that is coloured by the samples being male or female like this:

[[File:Metaphlan_module4_sex.png|600px]]

As you can see there is no obvious separation in the data points when coloured by sex of the person giving the sample. 

Now change the group field (top right) to “body_site” and the PCA will be coloured according to that grouping instead.  

Q10) Do you see any separation in the samples when coloured by body site? If so, describe the body sites that separate and which PC axis differentiates these samples.

Now lets test what is significantly different between the groups at the Genus rank. 

Under the “Multiple groups” dialog on the left, check that ANOVA is being used as the statistical test, and select “Benjamini-Hochberg FDR” for the multiple test correction. The box at bottom will say how the “Number of active features” using these set of statistics and should indicate that there are 12 organisms at the genus level that are statistically different in their means across the three body sites. 

[[File:STAMP_Stats.png]]

Now lets look at visualizations of these different taxonomic groups. 

Change the drop-down box that says "PCA plot" to "Bar plot".

A new window will show a list of all the Genera. You can filter this list to just the 12 Genera passing the statistical test by clicking the "Show only active features" box below the list of Genera. 

You can also order the list of Genera by the most significant by clicking on the "corrected p-value" header. Now if you click on the feature "g_Granulicatella" you should see a bar chart representing the relative abundance for each sample for this Genera along with the mean value for the group indicated by a horizontal line. The plot should look like this:

[[File:Metaphlan_module4_genera_bar.png|600px]]

Explore each of the different visualizations by changing "Bar plot" to each of the other options available (e.g. Heatmap plot, Post-hoc plot, and Box plot).

Q11) Create a box plot for the family Streptococcaceae (Change "Profile Level" to "Family" in the top left). Save the box plot as a .png image by using the File->Save plot feature in STAMP.