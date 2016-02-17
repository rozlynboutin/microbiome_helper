Our script chimeraFilter.pl wraps [usearch](http://drive5.com/usearch/) (v6.1), specifically the [uchime algorithm](http://www.drive5.com/usearch/manual/uchime_algo.html), to remove chimeric reads. 

Here is an example command:

    chimeraFilter.pl -type 1 -db /usr/local/db/single_strand/Bacteria_RDP_trainset15_092015.udb fasta_files/* 