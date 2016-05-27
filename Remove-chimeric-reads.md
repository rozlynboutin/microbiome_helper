Our script chimera_filter.pl wraps [VSEARCH](https://github.com/torognes/vsearch), which implements the [uchime algorithm](http://www.drive5.com/usearch/manual/uchime_algo.html), to remove chimeric reads. 

Here is an example command:

    chimera_filter.pl -type 1 -db /usr/local/db/single_strand/Bacteria_RDP_trainset15_092015.fa fasta_files/*

Where "-type 1" means that any reads clearly called as chimeric AND reads that are ambiguous are filtered out. 

Note that a DB file needs to be input as well. You can download the versions we use from our [requirements page](https://github.com/mlangill/microbiome_helper/wiki/Requirements). 

Note that it is possible that the settings of "mindiv" and "minh" (see http://www.drive5.com/usearch/manual/UCHIME_score.html) could have significant effects on results. However, so far we have found that small adjustments in these parameters has only a minor effect on sensitivity and specificity when running chimera checking for 16S sequences.

If you'd like to use USEARCH v6.1 (this is not open-source, but you can look into the license for a 32-bit version at the [USEARCH website](http://www.drive5.com/usearch/)) instead of VSEARCH, you can use the older version of our script called "chimera_filter_usearch61.pl" (the options are mainly the same).
 
Options:

* -h, --help <br>
   Displays the entire help documentation.

* -v, --version <br>
   Displays version number and exits.

* --type <[0|1]> <br>
   Non-chimeric output type, either only sequences that are clearly non-chimeric (1) or all sequences that are not called as chimeric ( 0 - includes borderline sequences, "?" in uchime output).

* --mindiv <float> <br>
   Min % divergence between query and target sequence (default 1.5, note that this differs from the uchime default of 0.8).

* --minh <float> <br>
   Min score to be called as chimeric (default 0.2, note that this differs from the uchime default of 0.28).

* -o, --out_dir <file> <br>
   Output directory for filtered fastq files. Default is "non_chimeras".

* --thread <# of CPUs> <br>
   Using this option without a value will use all CPUs on machine, while giving it a value will limit to that many CPUs. Without option only one CPU is used.

* --keep  
   Flag to indicate that temporary log files should not be removed (useful for troubleshooting). Also, will prevent the "nonchimera" and "unclear" specific fastas from being removed when type == 0.
   
* -log <file> <br>
   The location to write the log file.

* -db, --database <file> <br>
   Database of 16S sequences to use as a reference (UDB or FASTA file).