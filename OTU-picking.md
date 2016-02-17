We use [QIIME](http://qiime.org) scripts for OTU picking.

The first step is to create a QIIME mapping file (as described here: http://qiime.org/documentation/file_formats.html).

The bare-bones mapping file can be created with this command:

    create_qiime_map.pl non_chimeras/* > map.txt

Then you will need to combine all of the FASTA files into a single file (with sample IDs added to each header line):

    add_qiime_labels.py -i non_chimeras/ -m map.txt -c FileInput -o combined_fasta

The next script, pick_open_reference_otus.py, takes in parameters from a "parameter file" as well as on the command line. This is because the script is a wrapper for several other QIIME scripts and you can specify the parameters for these other scripts within the parameter file. We usually just specify two options for the pick_otus.py script:

    echo "pick_otus:threads 4" >> clustering_params.txt
    echo "pick_otus:sortmerna_coverage 0.8" >> clustering_params.txt

These parameters mean that we want to thread the pick_otus.py script over 4 CPUs and set the minimum percent query alignable coverage to be 80% (for SortMeRNA, as specified below).


Finally we can run the actual OTU picking step, specifying that we will be using SortMeRNA for reference picking and sumaclust for _de novo_ OTU picking (~24 hours):

    pick_open_reference_otus.py -i $PWD/combined_fasta/combined_seqs.fna -o $PWD/clustering/ -p $PWD/clustering_params.txt -m sortmerna_sumaclust -s 0.1 -v --min_otu_size 1 

After OTU picking it can be difficult to distinguish low frequency OTUs from noise. We filter out OTUs that supported by less than 0.1% reads (this is because Illumina reported that 0.1% of reads are expected to be bleed-through from previous runs on an Illumina MiSeq). 
Note that we retain singletons (OTUs identified by 1 read) for this step so that the threshold for 0.1% of reads can be correctly calculated in the next step.
