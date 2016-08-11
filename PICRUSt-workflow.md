(1) First reduce OTU table to just the reference OTUs.

        filter_otus_from_otu_table.py -i final_otu_tables/otu_table.biom -o final_otu_tables/closed_otus.biom --negate_ids_to_exclude -e /usr/local/lib/python2.7/dist-packages/qiime_default_reference/gg_13_8_otus/rep_set/97_otus.fasta

2. Convert from BIOM encoding from HDF5 to JSON.

        biom convert -i final_otu_tables/closed_otus.biom -o final_otu_tables/closed_otus_json.biom --to-json --table-type "OTU table"

(Temporary fix) Install BIOM 1.3.1 and temporarily change your PYTHONPATH

        pip install --target ~/lib biom-format==1.3.1
        export PYTHONPATH=~/lib/

3. PICRUSt: normalize OTU table by predicted 16S copy numbers. NOTE: PICRUSt has not been updated yet for BIOM 2.1. Therefore you must change to a python environment with biom 1.3.1 for the next 3 commands).


        normalize_by_copy_number.py -i final_otu_tables/closed_otus_json.biom -o final_otu_tables/closed_otus_norm.biom

4. PICRUSt: Predict KOs .... this is magical :)

        predict_metagenomes.py -i final_otu_tables/closed_otus_norm.biom -o final_otu_tables/ko.biom

5. PICRUSt: Collapse KOs into KEGG Pathways.

        categorize_by_function.py -i final_otu_tables/ko.biom -l 3 -c KEGG_Pathways -o ko_L3.biom

(Temporary Fix) Revert back to newer BIOM version by exiting and logging back into the shell or:

        export PYTHONPATH=''

6. Convert BIOM to STAMP format.

        biom_to_stamp.py -m KEGG_Pathways ko_L3.biom > ko_L3.spf
