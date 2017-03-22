1) First reduce OTU table to just the reference OTUs.

        filter_otus_from_otu_table.py -i final_otu_tables/otu_table.biom -o final_otu_tables/closed_otus.biom --negate_ids_to_exclude -e /usr/local/lib/python2.7/dist-packages/qiime_default_reference/gg_13_8_otus/rep_set/97_otus.fasta

2) PICRUSt: normalize OTU table by predicted 16S copy numbers. 


        normalize_by_copy_number.py -i final_otu_tables/closed_otus.biom -o final_otu_tables/closed_otus_norm.biom

3) PICRUSt: Predict KOs .... this is magical :)

        predict_metagenomes.py -i final_otu_tables/closed_otus_norm.biom -o final_otu_tables/ko.biom

4) PICRUSt: Collapse KOs into KEGG Pathways.

        categorize_by_function.py -i final_otu_tables/ko.biom -l 3 -c KEGG_Pathways -o final_otu_tables/ko_L3.biom

5) Convert BIOM to STAMP format.

        biom_to_stamp.py -m KEGG_Pathways final_otu_tables/ko_L3.biom > final_otu_tables/ko_L3.spf


**Note:**
If you are using PICRUSt 1.0.0 then you will need to make sure you are using BIOM 1.3.1 (and not the newer BIOM 2.1.\*). 
**This is not an issue with the newer PICRUSt versions.**  

To get around this you can:

Install BIOM 1.3.1 and temporarily change your PYTHONPATH (after #2 above)

        pip install --target ~/lib biom-format==1.3.1
        export PYTHONPATH=~/lib/

And then revert back to newer BIOM version by exiting and logging back into the shell or (after #5):

        export PYTHONPATH=''

  
  
Also, if you are using v1.0.0, you will need to convert to JSON BIOM format before running PICRUSt:

    biom convert -i final_otu_tables/closed_otus.biom -o final_otu_tables/closed_otus_json.biom --to-json --table-type "OTU table"

