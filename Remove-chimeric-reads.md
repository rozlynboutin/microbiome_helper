Our script chimeraFilter.pl wraps [usearch](http://drive5.com/usearch/) (v6.1), specifically the [uchime algorithm](http://www.drive5.com/usearch/manual/uchime_algo.html), to remove chimeric reads. 

Here is an example command:

    chimeraFilter.pl -type 1 -db /usr/local/db/single_strand/Bacteria_RDP_trainset15_092015.udb fasta_files/*

Where "-type 1" means that any reads clearly called as chimeric AND reads that are ambiguous are filtered out. 

Note that a DB file needs to be input as well. If you'd like to use the [UDB](http://www.drive5.com/usearch/manual/udb_files.html) format rather than FASTA then you'll need to use the "-makeudb_usearch" function of usearch v6.1 (the same usearch version as used for chimera checking). 

Options:

* -h, --help <br>
   Displays the entire help documentation.

* -v, --version <br>
   Displays version number and exits.

* -type <[0|1]> <br>
   Non-chimeric output type, either only sequences that are clearly non-chimeric (1) <br>
   or <br>
   all sequences that are not called as chimeric ( 0 - includes borderline sequences, "?" in uchime output).

* -mindiv <float> <br>
   Min % divergence between query and target sequence (default 1.5, note that this differs from the uchime default of 0.8).

* -minh <float> <br>
   Min score to be called as chimeric (default 0.2, note that this differs from the uchime default of 0.28).

* -o, --out_dir <file> <br>
   Output directory for filtered fastq files. Default is "non_chimeras".

* -thread <# of CPUs> <br>
   Using this option without a value will use all CPUs on machine, while giving it a value will limit to that many CPUs. Without option only one CPU is used.

* -log <file> <br>
   The location to write the log file.

* -db, --database <file> <br>
   Database of 16S sequences to use as a reference (UDB or FASTA file).
