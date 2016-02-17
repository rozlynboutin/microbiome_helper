run_metaphlan2.pl is used to wrap [MetaPhlAn2](https://bitbucket.org/biobakery/metaphlan2) for taxonomic composition:

    run_metaphlan2.pl -p 4 -o metaphlan_taxonomy.txt screened_reads/*

This taxonomy file can then be converted to a [STAMP](http://kiwi.cs.dal.ca/Software/STAMP) profile file with:

    metaphlan_to_stamp.pl metaphlan_taxonomy.txt > metaphlan_taxonomy.spf