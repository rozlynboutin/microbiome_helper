We have packaged two tools with the Microbiome Helper VBox that can link OTUs with phenotype-based functions: [BugBase](https://bugbase.cs.umn.edu/index.html) and [FAPROTAX](http://www.zoology.ubc.ca/louca/FAPROTAX/lib/php/index.php?section=Home). If you use either of these tools make sure you check these links for details and instructions for how to cite each tool.  
  
If you are more interested in predicting gene-based functions then you should check out the PICRUSt [workflow](https://github.com/mlangill/microbiome_helper/wiki/PICRUSt-workflow) and/or [tutorial](https://github.com/mlangill/microbiome_helper/wiki/PICRUSt-tutorial) we have written.
  
1) BugBase is a tool that links OTUs with high-level phenotypes such as whether they are gram positive or negative, whether they form biofilms, are pathogenic, etc. An in-depth description is available on the [BugBase website](https://bugbase.cs.umn.edu/index.html). Note that for most users you'll likely be able to use [BugBase online on the authors' website](https://bugbase.cs.umn.edu/upload.html). 
  
To try out BugBase on the Virtual Box you can run the demo data (you can view all options with run.bugbase.r -h):
    
    cd /home/shared/predict_pheno/BugBase_demo  
    run.bugbase.r -i HMP_s15.txt -m HMP_map.txt -c HMPBODYSUBSITE -o output   

BugBase requires a closed OTU table (meaning that _de novo_ OTUs should be excluded). If you ran open-reference OTU picking you can exclude _de novo_ OTUs with a command like this:

        filter_otus_from_otu_table.py -i PATH/TO/OTU/TABLE/otu_table.biom -o ./closed_otus.biom --negate_ids_to_exclude -e /usr/local/lib/python2.7/dist-packages/qiime_default_reference/gg_13_8_otus/rep_set/97_otus.fasta

