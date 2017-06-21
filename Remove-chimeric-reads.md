### Reference-based chimera checking

Our script chimera_filter.pl wraps [VSEARCH](https://github.com/torognes/vsearch), which implements the [uchime algorithm](http://www.drive5.com/usearch/manual/uchime_algo.html), to remove chimeric reads. 

Here is an example command:

```
chimera_filter.pl -type 1 -db /home/shared/rRNA_db/RDP_trainset16_022016.fa fasta_files/*
```

Where "-type 1" means that any reads clearly called as chimeric AND reads that are ambiguous are filtered out. 

Note that a DB file needs to be input as well. You can download the versions we use from our [requirements page](https://github.com/mlangill/microbiome_helper/wiki/Requirements). 

Note that it is possible that the settings of "mindiv" and "minh" ([see here for details]( http://www.drive5.com/usearch/manual/UCHIME_score.html)) could have significant effects on results. However, so far we have found that small adjustments in these parameters has only a minor effect on sensitivity and specificity when running chimera checking for 16S sequences.

If you'd like to use USEARCH v6.1 (this is not open-source, but you can look into the license for a 32-bit version at the [USEARCH website](http://www.drive5.com/usearch/)) instead of VSEARCH, you can use the older version of our script called "chimera_filter_usearch61.pl" (the options are mainly the same).
 
Options:

* -h, --help <br>
   Displays the entire help documentation.

* -v, --version <br>
   Displays version number and exits.

* --type <[0|1]> <br>
   Non-chimeric output type, either only sequences that are clearly non-chimeric (1) or all sequences that are not called as chimeric ( 0 - includes borderline sequences, "?" in uchime output).

* --mindiv < float > <br>
   Min % divergence between query and target sequence (default 1.5, note that this differs from the uchime default of 0.8).

* --minh < float > <br>
   Min score to be called as chimeric (default 0.2, note that this differs from the uchime default of 0.28).

* -o, --out_dir <file> <br>
   Output directory for filtered fastq files. Default is "non_chimeras".

* --thread <# of CPUs> <br>
   Using this option without a value will use all CPUs on machine, while giving it a value will limit to that many CPUs. Without option only one CPU is used.

* --keep  
   Flag to indicate that temporary log files should not be removed (useful for troubleshooting). Also, will prevent the "nonchimera" and "unclear" specific fastas from being removed when type == 0.
   
* -log <file> <br>
   The location to write the log file. Default is "chimera_filter.log".

* -db, --database <file> <br>
   Database of 16S sequences to use as a reference (FASTA file).

### _De novo_ chimera checking

If you would prefer to run _de novo_ chimera checking this is also simple to do with VSEARCH, based on the UCHIME _de novo_ algorithm. We don't have a wrapper script to run the commands, but they can be easily run with [GNU parallel](https://www.gnu.org/software/parallel/) ([see our basic tutorial for basic usage](https://github.com/mlangill/microbiome_helper/wiki/Quick-Introduction-to-GNU-Parallel)). The catch is that there will be raw logfiles for each FASTA rather than a summary logfile as above.

Before running _de novo_ chimera detection it's important to run dereplication of your reads with VSEARCH. If you skip this step then you'll get 0 chimeric reads identified (as described on the VSEARCH google group [here](https://groups.google.com/forum/#!topic/vsearch-forum/roWz8_nw2Tk) and [here](https://groups.google.com/forum/#!topic/vsearch-forum/dZsBbU1h_Pw)). Dereplication is required since identifying chimeras is based on finding possible parents that by default have at least twice the abundance of the chimeras (note that this option and several other chimera detection options can be set by the user). You'll need to rereplicate your sequences before continuing with the standard Microbiome Helper pipeline.

First make the output folders.

```
mkdir fasta_files_derep
mkdir non_denovo_chimeras
mkdir non_denovo_chimeras_rerep
```

Then run dereplication with GNU parallel.
```
parallel -j 4 'vsearch --derep_fulllength {} --sizeout --output fasta_files_derep/{/.}.derep.fasta' ::: fasta_files/*fasta
```

Run the uchime_denovo algorithm with GNU parallel.
```
parallel --eta -j 4 'vsearch --uchime_denovo {} --nonchimeras non_denovo_chimeras/{/.}.nonchimera.fasta 2> non_denovo_chimeras/{/.}.nonchimera.log' ::: fasta_files_derep/*.derep.fasta
```

Note that the logfile for each FASTA will be in the same output directory as the chimera-filtered FASTAs.

Finally, run _rereplication_ of your sequences so that you can continue with the standard Microbiome Helper workflow.

```
parallel -j 4 'vsearch --rereplicate {} --output non_denovo_chimeras_rerep/{/.}.rerep.fasta' ::: non_denovo_chimeras/*fasta
```

The FASTA files to carry on to downstream steps will then be in _non\_denovo\_chimeras\_rerep_.