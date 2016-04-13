Note that these commands have not yet been tested and we are still working on the 18S version of the pipeline.

The key differences between 18S and 16S workflows are the reference databases used for chimera checking and OTU picking. 

Another difference is that there is much higher variation in 18S rRNA coding region lengths than in 16S rRNA. For this reason we recommend not using 400 bp as the minimum length cut-off for our readFilter.pl script. A reasonable choice is 200 bp:

    readFilter.pl -f CYGCGGTAATTCCAGCTC -r CRAAGAYGATYAGATACCRT -thread 4 -q 30 -p 90 -l 200 stitched_reads/*.assembled.*

Also, it is possible that some of the lengthier 18S rRNA genes wont be assembled properly. It is a good idea to check the proportion of unassembled forward reads to make sure that you are not throwing out too much of your data. One approach could be to include all unassembled forward reads for downstream analyses, but we haven't explored how the quality cut-off and OTU-picking would have to be changed, so we don't recommend that yet.

This command should be used for the chimera checking step:

    chimeraFilter.pl -thread 4 -type 1 -db /home/shared/rRNA_db/Eukaryota_SILVA_123_SSURef_Nr99_tax_silva_U-replaced.udb fasta_files/*

Also, these commands should be used for the OTU picking step:

    echo "pick_otus:threads 4" >> clustering_params.txt
    echo "pick_otus:sortmerna_coverage 0.8" >> clustering_params.txt
    echo "pick_otus:similarity 0.99" >> clustering_params.txt
    echo "align_seqs:template_fp /home/shared/rRNA_db/Silva_119_rep_set90_aligned_18S.fna" >> clustering_params.txt 
    echo "assign_taxonomy:id_to_taxonomy_fp /home/shared/rRNA_db/Silva_119_rep_set99_18S_taxonomy_7_levels.txt" >> clustering_params.txt
    echo "assign_taxonomy:reference_seqs_fp /home/shared/rRNA_db/Silva_119_rep_set99_18S.fna" >> clustering_params.txt

    pick_open_reference_otus.py -i $PWD/combined_fasta/combined_seqs.fna -o $PWD/clustering/ -p $PWD/clustering_params.txt -m sortmerna_sumaclust -s 0.1 -v --min_otu_size 1 -r /home/shared/rRNA_db/Silva_119_rep_set99_18S.fna

The current 18S reference database we are using has some naming conflicts when used with STAMP, which we are temporarily getting around with the below commands when converting from BIOM to STAMP:

    biom convert -i final_otu_tables/otu_table.biom -o final_otu_tables/otu_table_w_tax.txt --to-tsv --header-key taxonomy

    TmpFixBIOM.pl final_otu_tables/otu_table_w_tax.txt  > final_otu_tables/otu_table_w_tax_cleaned.txt

    biom convert -i  final_otu_tables/otu_table_w_tax_cleaned.txt -o final_otu_tables/otu_table_cleaned.biom --to-hdf5 --process-obs-metadata taxonomy --table-type="OTU table"

    biom_to_stamp.py -m taxonomy final_otu_tables/otu_table_cleaned.biom >final_otu_tables/otu_table_w_tax_cleaned.spf

    sed -i 's/D_4__Chlorarachniophyta\tD_5__NOR26/D_4__Chlorarachniophyta\tD_5__NOR26_Chlorarachniophyta/g’  final_otu_tables/otu_table_w_tax_cleaned.spf
