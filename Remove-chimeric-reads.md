Our script chimeraFilter.pl wraps [usearch](http://drive5.com/usearch/) (v6.1), specifically the [uchime algorithm](http://www.drive5.com/usearch/manual/uchime_algo.html), to remove chimeric reads. 

Here is an example command:

    chimeraFilter.pl -type 1 -db /usr/local/db/single_strand/Bacteria_RDP_trainset15_092015.udb fasta_files/*

Where "-type 1" means that any reads clearly called as chimeric AND reads that are ambiguous are filtered out. Note that a DB file needs to be input as well and if you'd like to use the [UDB](http://www.drive5.com/usearch/manual/udb_files.html) format rather than FASTA then you'll need to use the "-makeudb_usearch" function of usearch v6.1 (the same usearch version as used for chimera checking). 