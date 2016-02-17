We use [QIIME](http://qiime.org) scripts for creating diversity plots. 

The first step is to create a summary of the final BIOM file that was created after filtering low confidence OTUs:

    biom summarize-table -i clustering/otu_table_high_conf.biom -o clustering/otu_table_high_conf_summary.txt

You can get more info on biom (i.e. biom-format) commands here: http://biom-format.org

We are especially interested in the sorted list of read counts under the "Counts/sample detail:" heading. This is because the next step is to normalize the OTU table to the same sample depth ("X" below). Ideally you would normalize to the sample with the lowest depth, however you might be throwing out too much information if that sample's depth is too low. This is how the command is run (you first need to make a directory called "final_otu_tables"):
    
    mkdir final_otu_tables
    single_rarefaction.py -i clustering/otu_table_high_conf.biom -o final_otu_tables/otu_table.biom -d X


Other normalization methods such as [DESeq2](https://bioconductor.org/packages/release/bioc/html/DESeq2.html) can be used instead if you don't want to throw out too much data (you should run "normalize_table.py -h" for more details): 
   
    normalize_table.py -i clustering/otu_table_high_conf.biom -a DESeq2 -z -o final_otu_tables/otu_table_deseq2_norm_no_negatives.biom

Using DESeq2 is still not mainstream, although that could be changing. 