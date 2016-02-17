Run MetaPhlAn2 for taxonomic composition.

    run_metaphlan2.pl -p 4 -o metaphlan_taxonomy.txt screened_reads/*

Convert from MetaPhlAn to STAMP profile file.

    metaphlan_to_stamp.pl metaphlan_taxonomy.txt > metaphlan_taxonomy.spf
