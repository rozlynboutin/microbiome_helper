We have packaged two tools with the Microbiome Helper VBox that can link OTUs with phenotype-based functions: [BugBase](https://bugbase.cs.umn.edu/index.html) and [FAPROTAX](http://www.zoology.ubc.ca/louca/FAPROTAX/lib/php/index.php?section=Home). If you use either of these tools make sure you check these links for details and instructions for how to cite each tool.  
  
If you are more interested in predicting gene-based functions then you should check out the PICRUSt [workflow](https://github.com/mlangill/microbiome_helper/wiki/PICRUSt-workflow) and/or [tutorial](https://github.com/mlangill/microbiome_helper/wiki/PICRUSt-tutorial) we have written.
  
### BugBase   
    
BugBase is a tool that links OTUs with high-level phenotypes such as whether they are gram positive or negative, whether they form biofilms, are pathogenic, etc. An in-depth description is available on the [BugBase website](https://bugbase.cs.umn.edu/index.html). Note that for most users you'll likely be able to use [BugBase online on the authors' website](https://bugbase.cs.umn.edu/upload.html). 
  
To try out BugBase on the Virtual Box you can run the demo data (you can view all options with run.bugbase.r -h):
    
    cd /home/shared/predict_pheno/BugBase_demo  
    run.bugbase.r -i HMP_s15.txt -m HMP_map.txt -c HMPBODYSUBSITE -o output   
  
In the folder "output" there will be a number of summary tables and plots which you can check out.  
  
The demo data has already been prepped in advance, but you might need a few extra steps to prepare your own data for BugBase. Firstly, BugBase requires a closed OTU table (meaning that _de novo_ OTUs should be excluded). If you ran open-reference OTU picking you can exclude _de novo_ OTUs with a command like this:

        filter_otus_from_otu_table.py -i otu_table.biom -o closed_otus.biom --negate_ids_to_exclude -e /usr/local/lib/python2.7/dist-packages/qiime_default_reference/gg_13_8_otus/rep_set/97_otus.fasta
  
You will also need to convert the BIOM file to JSON format. The below command is probably what you'll need, but detailed instructions for how to do this are on the [BIOM website](http://biom-format.org/documentation/biom_conversion.html):
  
    biom convert -i otu_table.biom -o otu_table_json.biom --table-type="OTU table" --to-json
   
  
### FAPROTAX

Functional Annotation of Prokaryotic Taxa (FAPROTAX) is a database that maps prokaryotic taxa with functions reported in the literature. See the [FAPROTAX website](http://www.zoology.ubc.ca/louca/FAPROTAX/lib/php/index.php?section=Home) for details. If you want to make changes to the database be sure to check out the [license agreement](http://www.zoology.ubc.ca/louca/FAPROTAX/lib/php/index.php?section=License). 

The script collapse_table.py can be used for linking your OTUs of interest with this database (which is "FAPROTAX.txt").
  
To learn how to do this you can try the two example commands described in the documentation:  
  
    cd /home/shared/predict_pheno/FAPROTAX_1.0/example_input
  
    collapse_table.py -i otu_table.tsv -o functional_table.tsv -g ../FAPROTAX.txt -c '#' -d 'taxonomy' --omit_columns 0 --column_names_are_in last_comment_line -r report_example1.txt -n columns_after_collapsing -v

    collapse_table.py -i otu_table.biom -o functional_table.biom -g ../FAPROTAX.txt -r report_example2.txt -n columns_after_collapsing -v --collapse_by_metadata 'taxonomy'

Be sure to cite the FAPROTAX paper if you use it:  
_Louca, S., Parfrey, L.W., Doebeli, M. (2016) - Decoupling function and taxonomy in the global ocean microbiome. Science 353:1272-1277_.