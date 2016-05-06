## Introduction

This tutorial is set-up to walk you through the process of determining the taxonomic and functional composition of several metagenomic samples. It covers the use of [Metaphlan2](http://huttenhower.sph.harvard.edu/metaphlan2) (for taxonomy), [HUMAnN](https://huttenhower.sph.harvard.edu/humann) (for functional) and [STAMP](http://kiwi.cs.dal.ca/Software/STAMP) (for visualization and statistical evaluation).  

Throughout the tutorial, there are several questions to ensure that you are understanding the process (and not just copy and pasting). You can check the [answer sheet](Metagenomics-Tutorial-Answers) after you have answered them yourself.

This lab component will use samples collected and sequenced through the Human Microbiome Project (HMP).  We will be analyzing samples collected from three oral sites including subgingival, supragingival, and the tongue. These samples have been downloaded for you already from the HMP-DACC (http://hmpdacc.org/HMIWGS/all/) and have been randomly subsampled to a much smaller number of reads to allow faster computation time. Real data will require more waiting time between each step!

**Author**: Morgan Langille

**Contributions by**: Gavin Douglas

**First Created**: Summer 2015 for CBW Metagenomics

**Last Edited**: Spring 2016 for CCBC, Toronto

## Requirements

* Basic unix skills (This is a good introductory tutorial: http://korflab.ucdavis.edu/bootcamp.html)
* [Microbiome Helper VirtualBox] (MicrobiomeHelper-Virtual-Box) (install it and ensure it is working)
* [Tutorial Data](https://www.dropbox.com/s/l87mmmjs0fr85ir/hmp_metagenomics.zip?dl=1)

## Initial Setup
* Open this tutorial within the Microbiome Helper VirtualBox
* Download the tutorial data, save it to the Desktop (within Ubuntu), and extract the files. 

## Main Lab Steps
### Explore Samples

Open a terminal/console and change to the directory containing the tutorial data.

    cd ~/Desktop/hmp_metagenomics

The tutorial data consists of a "map file" containing information about each of the samples called **hmp_map.txt**, along with the sequence data for each sample being contained within a separate fastq file within the directory **fastq**.

Lets take a look at the **hmp_map.txt** using _less_ (type q to quit out of the program when done)

    less hmp_map.txt

You will have noticed that this map file contains three columns with the first being sample ids, the middle column being the body site, and the third being the sex of the person. 

Remember you can count the lines of a file using 'wc -l'. For example,

    wc -l hmp_map.txt

**Q1)** How many samples are there in total (you can either look at the hmp_map.txt file or count the fastq files)?

**Q2)** How many samples are from each of the different sample sites?

**Q3)** How many sequences are there in each of the samples? Remember there are 4 lines in a fastq file for each sequence. 

### Taxonomic Profiling with Metaphlan2
We will use Metaphlan2 to determine the taxonomic composition of each sample.  As with all other tools, Metaphlan2 has already been installed within the Microbiome Helper Virtualbox. 

Similar to other bioinformatic programs you can get the full help for the program by typing the ‘-h’ option after the program name:
 
    metaphlan2.py -h

As you can see there are quite a few options. We won’t explore all of these in this tutorial, but it may be worth reading on your own time. 

Find the option which tells you the version of Metaphlan2 you are using. 

**Q4)** What version of Metaphlan are you using?

### Running Metaphlan

Now we are going to run metaphlan on the sample "SRS015044". 

First change back to your module 3 directory:

    cd ~/Desktop/hmp_metagenomics

Now we are going to run metaphlan on a single example with the following (long) command:
    
    metaphlan2.py --mpa_pkl /usr/local/prg/metaphlan2/db_v20/mpa_v20_m200.pkl --input_type fastq --bowtie2db /usr/local/prg/metaphlan2/db_v20/mpa_v20_m200 --no_map -o SRS015044.txt fastq/SRS015044.fastq

The command will run for 1-2 minutes on this single sample.

The command line parameters are:
* `--mpa_pkl /usr/local/prg/metaphlan2/db_v20/mpa_v20_m200.pkl `: Indicates where the Metaphlan marker database is located which contains metadata information about each of the markers
* `--input_type fastq`: Indicates that our input files are in fastq format. (If we wanted to use fasta files as input then we would change this to 'fasta').
* `--bowtie2db /usr/local/prg/metaphlan2/db_v20/mpa_v20_m200`: This indicates the location of the marker database formatted for use with bowtie2
* `--no_map` We use this flag to prevent metaphlan from storing the intermediate bowtie output files. 
* `-o SRS015044.txt`: Indicates the name of output file that metaphlan will use to write results to.
* `fastq/SRS015044.fastq`: Metaphlan takes the input file containing our metagenomic reads as the last argument

Now we are going to run metaphlan on another sample "SRS015893". 

Note: If you have altered the settings off your virtualbox  to use more cores, than you can make this command faster by adding the following option (assuming you have two cores enabled) `-–nproc 2`:

    metaphlan2.py --mpa_pkl /usr/local/prg/metaphlan2/db_v20/mpa_v20_m200.pkl --input_type fastq --bowtie2db /usr/local/prg/metaphlan2/db_v20/mpa_v20_m200 --no_map -o SRS015893.txt fastq/SRS015893.fastq 

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

```
k__Bacteria|p__Fusobacteria|c__Fusobacteriia|o__Fusobacteriales|f__Leptotrichiaceae	3.2895
k__Bacteria|p__Fusobacteria|c__Fusobacteriia|o__Fusobacteriales|f__Leptotrichiaceae|g__Leptotrichia	2.58678
k__Bacteria|p__Fusobacteria|c__Fusobacteriia|o__Fusobacteriales|f__Leptotrichiaceae|g__Leptotrichiaceae_unclassified	0.70272
k__Bacteria|p__Fusobacteria|c__Fusobacteriia|o__Fusobacteriales|f__Leptotrichiaceae|g__Leptotrichia|s__Leptotrichia_unclassified	1.98806
k__Bacteria|p__Fusobacteria|c__Fusobacteriia|o__Fusobacteriales|f__Leptotrichiaceae|g__Leptotrichia|s__Leptotrichia_buccalis	0.49975
k__Bacteria|p__Fusobacteria|c__Fusobacteriia|o__Fusobacteriales|f__Leptotrichiaceae|g__Leptotrichia|s__Leptotrichia_wadei	0.05456
k__Bacteria|p__Fusobacteria|c__Fusobacteriia|o__Fusobacteriales|f__Leptotrichiaceae|g__Leptotrichia|s__Leptotrichia_hofstadii	0.0444
k__Bacteria|p__Fusobacteria|c__Fusobacteriia|o__Fusobacteriales|f__Leptotrichiaceae|g__Leptotrichia|s__Leptotrichia_buccalis|t__GCF_000023905	0.49975
k__Bacteria|p__Fusobacteria|c__Fusobacteriia|o__Fusobacteriales|f__Leptotrichiaceae|g__Leptotrichia|s__Leptotrichia_wadei|t__Leptotrichia_wadei_unclassified	0.05456
k__Bacteria|p__Fusobacteria|c__Fusobacteriia|o__Fusobacteriales|f__Leptotrichiaceae|g__Leptotrichia|s__Leptotrichia_hofstadii|t__GCF_000162955	0.0444
```

You can see that 3.2895% of the metagenome is predicted from organisms in the family Leptotrichiaceae. Of those ~2.6% are assigned to the genus _Leptotrichia_, while ~0.7% are assigned to _g__Leptotrichiaceae_unclassified_ which means metaphlan doesn't know what genus to actually call them. 

**Q5)** What is the relative abundance of organisms unclassified at the species level for the genus _Neisseria_ in sample SRS015044? (Remember to use the grep command)
 
**Q6)** What phylum has the highest relative abundance in sample SRS015044? And SRS015893?

Now try running metaphlan using the same options as above but for sample **SRS097871**. (Remember to change the input and output files)

### Merging Metaphlan Results 

Now lets combine all of the metaphlan output files into a single merged output file:

 /usr/local/prg/metaphlan2/utils/merge_metaphlan_tables.py SRS015044.txt SRS015893.txt SRS097871.txt > metaphlan_merged.txt

Note that this script 'merge_metaphlan_tables.py' takes one or more metaphlan output files as input and combines them into a single output file. The output file is indicated using the stdout redirect symbol '>' and is written in this case to **metaphlan_merged.txt**

The merged output file should look like this:
```
 ID      SRS015044       SRS015893       SRS097871
 #SampleID       Metaphlan2_Analysis     Metaphlan2_Analysis     Metaphlan2_Analysis
 k__Bacteria     100.0   100.0   100.0
 k__Bacteria|p__Actinobacteria   33.58989        23.49742        66.06867
 k__Bacteria|p__Actinobacteria|c__Actinobacteria 33.58989        23.49742        66.06867
 k__Bacteria|p__Actinobacteria|c__Actinobacteria|o__Actinomycetales      33.58989        22.83325        66.06867
 k__Bacteria|p__Actinobacteria|c__Actinobacteria|o__Actinomycetales|f__Actinomycetaceae  4.60715 22.27157        20.84159
```
Now each sample is listed as a different column within this output file. You can view this file again using 'less' or you can import it into your favourite spreadsheet program.

**Q7)** What phylum is present in one of the samples but is absent in the others? What sample is this phylum in? 

To get information about the individual samples you can look in the hmp_map.txt file using 'less':

 less hmp_metagenomics/hmp_map.txt

**Q8)** What body site is the sample taken from that contains the unique phylum from question 6?

### Running Metaphlan on a large number of samples using Microbiome Helper

Running metaphlan on more than a few samples can be tedious. 

To make this process simpler we are going to use a script I wrote as part of the package “Microbiome Helper” (https://github.com/mlangill/microbiome_helper).

Run metahplan on all of the sample using 'run_metaphlan2.pl' (this will take about 15 minutes):

    run_metaphlan2.pl -p 8 -o metaphlan_merged_all.txt hmp_metagenomics/fastq/*

This command uses the following options:
* '-p 8' runs metaphlan in parallel using 8 processors.
* '-o' is the name for the merged output file.
* ' hmp_metagenomics/fastq/*' indicates that all of the files within this directory will be used as input. 

On your screen you should see the commands that the microbiome helper script is running automatically for you:
```
 cat hmp_metagenomics/fastq/SRS014477.fastq | /usr/local/metaphlan2/metaphlan2.py  --input_type multifastq --mpa_pkl /usr/local/metaphlan2/db_v20/mpa_v20_m200.pkl --bt2_ps sensitive-local --min_alignment_len 50 --bowtie2db /usr/local/metaphlan2/db_v20/mpa_v20_m200 --no_map > ./metaphlan_out/SRS014477
 cat hmp_metagenomics/fastq/SRS011343.fastq | /usr/local/metaphlan2/metaphlan2.py  --input_type multifastq --mpa_pkl /usr/local/metaphlan2/db_v20/mpa_v20_m200.pkl --bt2_ps sensitive-local --min_alignment_len 50 --bowtie2db /usr/local/metaphlan2/db_v20/mpa_v20_m200 --no_map > ./metaphlan_out/SRS011343
 cat hmp_metagenomics/fastq/SRS019129.fastq | /usr/local/metaphlan2/metaphlan2.py  --input_type multifastq --mpa_pkl /usr/local/metaphlan2/db_v20/mpa_v20_m200.pkl --bt2_ps sensitive-local --min_alignment_len 50 --bowtie2db /usr/local/metaphlan2/db_v20/mpa_v20_m200 --no_map > ./metaphlan_out/SRS019129
```
**Q9)** Based on the commands being written to the screen, what directory are the metaphlan individual output files being stored in?

### STAMP

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

**Q10)** Do you see any separation in the samples when coloured by body site? If so, describe the body sites that separate and which PC axis differentiates these samples.

Now lets test what is significantly different between the groups at the Genus rank. 

Under the “Multiple groups” dialog on the left, check that ANOVA is being used as the statistical test, and select “Benjamini-Hochberg FDR” for the multiple test correction. The box at bottom will say how the “Number of active features” using these set of statistics and should indicate that there are 12 organisms at the genus level that are statistically different in their means across the three body sites. 

[[File:STAMP_Stats.png]]

Now lets look at visualizations of these different taxonomic groups. 

Change the drop-down box that says "PCA plot" to "Bar plot".

A new window will show a list of all the Genera. You can filter this list to just the 12 Genera passing the statistical test by clicking the "Show only active features" box below the list of Genera. 

You can also order the list of Genera by the most significant by clicking on the "corrected p-value" header. Now if you click on the feature "g_Granulicatella" you should see a bar chart representing the relative abundance for each sample for this Genera along with the mean value for the group indicated by a horizontal line. The plot should look like this:

[[File:Metaphlan_module4_genera_bar.png|600px]]

Explore each of the different visualizations by changing "Bar plot" to each of the other options available (e.g. Heatmap plot, Post-hoc plot, and Box plot).

**Q11)** Create a box plot for the family Streptococcaceae (Change "Profile Level" to "Family" in the top left). Save the box plot as a .png image by using the File->Save plot feature in STAMP.


### Determining Functional Composition with HUMAnN

We will now functionally annotate the metagenomes using HUMAnN. Note, that we will be using the same HMP samples used above.

## Running DIAMOND search against KEGG

Before running HUMAnN, it is first necessary run a similarity search against the KEGG database. This database is located at “/home/shared/kegg/diamond_db” on the Virtual Box. This is a reduced KEGG database that the authors of HUMAnN created and which they make available through personal correspondence. This similarity search can be done using various tools such as BLAST, USEARCH, RapSearch, etc. In this tutorial we will use a relatively new similarity search tool that is much faster and works well for large metagenomic datasets called [DIAMOND](http://ab.inf.uni-tuebingen.de/software/diamond/).

First we will make a directory to store our sequence search outputs:

    mkdir pre_humann

Now we will run DIAMOND on our metagenomic sequences for the sample SRS015044:
 
    diamond blastx -p 2 -d kegg/kegg.reduced -q hmp_metagenomics/fastq/SRS015044.fastq -a pre_humann/SRS015044

The options used in this command are:

* 'blastx': Tells DIAMOND to run in “blastx” mode meaning that we will search a nucleotide query against a protein database in all 6 frame translations (3 forward and 3 reverse).
* '-p 2': Indicates that DAMOND should use 2 threads to do the search.
*  '-d kegg/kegg.reduced' points at the KEGG database which has already been formatted for use with DIAMOND
* '-q hmp_metagenomics/fastq/SRS015044.fastq' is the input metagenomic sample
* '-a pre_humann/SRS015044' is the name of the output file (Note: DIAMOND appends a '.daa' automatically to the end of the output file name)
* Note: That the default e-value cutoff used by DIAMOND is 0.001.

The previous command is comparing 200k sequences vs. the KEGG database (~1.3 million sequences) and takes about 3 minutes. If we had used the NCBI BLASTX it would have taken a couple of days!

The  output from diamond is a special binary format that we can then turn into either SAM or BLAST tabular output with the latter being the default.

 diamond view -a pre_humann/SRS015044.daa -o pre_humann/SRS015044.txt

You can see the output using 'less':

 less pre_humann/SRS015044.txt

Each column tells us different information about the match:
 1	Query
 2	Subject
 3	% id
 4	alignment length
 5	Mistmatches
 6	gap openings
 7	q.start
 8	q.end
 9	s.start
 10	s.end
 11	e-value
 12	bit score

Now run DIAMOND for samples SRS015893 aagainst the KEGG database and convert output to BLAST tabular format.

 diamond blastx -p 4 -d kegg/kegg.reduced -q hmp_metagenomics/fastq/SRS015893.fastq -a pre_humann/SRS015893
 diamond view -a pre_humann/SRS015893.daa -o pre_humann/SRS015893.txt

Repeat the above commands once again for sample SRS097871.

===Running HUMAnN===
 
Now, we are going to run HUMAnN. To use human you are going to copy the entire program to your working directory. The human program is in “ ~/CourseData/refs/humann-0.99/”:

 cp -r ~/CourseData/refs/humann-0.99/ ./

Then move into the HUMAnN directory:
 
 cd humann-0.99

Now copy your blast output from your 3 samples into the “human-0.99/input” directory:

 cp ../pre_humann/SRS015044.txt ../pre_humann/SRS015893.txt ../pre_humann/SRS097871.txt ./input/

To being running human on just these 3 samples you use the “scons” command (using 4 cores):

 scons -j 4

A bunch of messages will pass on your screen and it should finish in ~2 minutes. All of the output is contained in the “humann-0.99/output” directory. To see a list of them type:
 
 ls output

There are MANY output files from human. The ones we care about are called:

* '''01b-hit-keg-cat.txt''' –> KEGG KOs
* '''04b-hit-keg-mpm-cop-nul-nve-nve.txt''' -> KEGG Modules
* '''04b-hit-keg-mpt-cop-nul-nve-nve.txt''' -> KEGG Pathways

These files contain relative abundances for each of these different functional classifications. You can look at the format of these using “less”. For example to look at the KEGG Pathways:

 less output/04b-hit-keg-mpt-cop-nul-nve-nve.txt

The output should look like this:

 ID      NAME    SRS015044-hit-keg-mpt-cop-nul-nve-nve   SRS015893-hit-keg-mpt-cop-nul-nve-nve   SRS097871-hit-keg-mpt-cop-nul-nve-nve
 InverseSimpson  InverseSimpson  63.9654 59.6469 62.4902
 Shannon Shannon 4.30945 4.25267 4.30185
 Pielou  Pielou  0.858922        0.847606        0.857407
 Richness        Richness        1       1       1
 ko00564 ko00564: Glycerophospholipid metabolism 0.00827982      0.00612454      0.00641725
 ko00680 ko00680: Methane metabolism     0.00564549      0.00529206      0.00536557
 ko00562 ko00562: Inositol phosphate metabolism  0.00241117      0.00268946      0.0048378
 ko00563 ko00563: Glycosylphosphatidylinositol(GPI)-anchor biosynthesis  5.69937e-05     0       0
 ko00561 ko00561: Glycerolipid metabolism        0.00496728      0.00468482      0.00548291

* The first line indicates that the first column is a short id for the KEGG Pathway, the second column is a longer description of the KEGG Pathway, and the remaining columns represent the sample identifiers. 
* The next 4 lines calculate different alpha-diversity metrics for each sample, but in general these are usually not that useful and can be ignored.
* From lines 6 onwards each row indicates the different relative abundance for each KEGG Pathway. Note that these values are small because they have been normalized so that each sample will sum to 1. 

Now, use less to look at the KO predictions and the KEGG Module predictions:

 less output/01b-hit-keg-cat.txt
 less output/04b-hit-keg-mpm-cop-nul-nve-nve.txt

Q1) Which of the three samples has the highest relative abundance of the KEGG Module: “M00319: Manganese/zinc/iron transport system”?

===Running all samples with Microbiome Helper===

So what about all of our samples? To do the DIAMOND searches for all 30 you could use a wrapper script provided by the microbiome_helper  package called run_pre_humann.pl. All 30 samples '''could be''' processed with a single command like:

 run_pre_humann.pl -p 8 -d kegg/kegg.reduced -o pre_humann hmp_metagenomics/fastq/*

However, this would take about 25 minutes to complete. The humann step would take an additional 10 minutes to complete.  

To make things easier the output for all 30 samples has been pre-computed and is located in  “hmp_metagenomics/humann_output”. 

If you browse the output using 'less' you can see that they are in the same format but with 30 columns representing the 30 samples:

 cd ~/workspace/module4
 less hmp_metagenomics/humann_output/04b-hit-keg-mpt-cop-nul-nve-nve.txt

You will notice that the output looks fairly messy because the terminal will automatically line wrap and it becomes hard to see where one line ends and the next begins. I often find the "cut" command useful to browse data like this. "cut" allows you to just look at particular columns from the data. For example:

 cut -f 1-5 hmp_metagenomics/humann_output/04b-hit-keg-mpt-cop-nul-nve-nve.txt | less

* '-f 1-5' indicates that we want to look at the first 5 columns. We could just have easily used '-f 1,3,10' (to view columns 1, 3 and 10) or '-f 1-3,20-' (to view columns 1 to 3 and columns 20 onwards).
* '| less' "pipes" the output from the "cut" command and lets us view the output one screen at a time using our 'less' tool.  

Now, we are going to use STAMP to visualize the humann output just like we did with the taxonomic data. 

First, we need to convert the humann output files file into STAMP format:

 humann_to_stamp.pl hmp_metagenomics/humann_output/04b-hit-keg-mpt-cop-nul-nve-nve.txt > pathways.spf
 humann_to_stamp.pl hmp_metagenomics/humann_output/04b-hit-keg-mpm-cop-nul-nve-nve.txt > modules.spf
 humann_to_stamp.pl hmp_metagenomics/humann_output/01b-hit-keg-cat.txt > kos.spf

You should now have the 3 files 'pathways.spf', 'modules.spf', and 'kos.spf' in the current directory. You can check to make sure they are there with 'ls'

 ls

Your output should look like this:

 ubuntu@ip-10-157-158-196:~/workspace/module4$ ls
 hmp_metagenomics  humann-0.99  kegg  kos.spf  modules.spf  pathways.spf  pre_humann
 ubuntu@ip-10-157-158-196:~/workspace/module4$

===STAMP with Humann Output===

Copy the 'pathways.spf', 'modules.spf', and 'kos.spf' along with the previously used "hmp_map.txt" file locally onto your computer. 

Load the kos.spf file along with the original hmp_map.txt file into STAMP. (File->Load)

Click on the "Two Groups" tab and choose "Benjamini-Hochberg FDR" for Multiple Test Correction. Look at the "Number of active features" to determine if there are any significant different features.

Q2) Using a two group test with multiple test correction applied are there any significant differences between male and female samples?

Change to a Multiple Group Test using the same multiple test correction and change the Group field (top right) to "body_site".

Q3) Are there any significant differences in KOs across the body sites? If so, how many?

Now load the "pathways.spf" file into STAMP using the same "hmp_map.txt" file. 

(Note: STAMP has a bug that will not load a new dataset on top of another, so you need to completely close and restart STAMP first before loading in a different dataset)

Q4) When comparing samples by body site, how many KEGG Pathways are significantly different when using a group test (ANOVA) with Benjamini-Hochberg FDR?

Choose "Box plot" instead of PCA plot, and then expand the list of KEGG Pathways so you can see the corrected p-value. Order the list by corrected p-value by clicking on that column header. 

[[File:STAMP_ordered_p-value.png]]

Q5) What is the most significantly different KEGG Pathway? What is the corrected p-value for this KEGG Pathway?

Compare the tongue samples to all other samples using a Two Group test. First select “tongue_dorsum” for Group 1 (on the left hand side) and then select “All other samples” as Group 2. Use the default Welch’s t-test with BH FDR.

Your PCA plot should look like this:

[[File:STAMP_tongue_pca.png|600px]]

Q6) How many KEGG Pathways are significantly different between the tongue and the plaque samples combined?

Create a "Bar plot" for "ko00020: Citrate cycle (TCA cycle)".

Q7) Is the relative abundance of “Citrate cycle (TCA cycle)” higher or lower in the tongue compared to the plaque samples?

Q8) Create an “Extended error bar” plot and save the image as a .png using the File->Save Plot option.