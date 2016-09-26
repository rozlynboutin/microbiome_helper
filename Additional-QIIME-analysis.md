*Once you have created an OTU table using the above pipeline, there are several statistcal tests and visualization methods available within QIIME. Not all of these are useful for all data types or projects. This list is meant more as a catalog of what is possible.*

* Get statistical signfance of category groupings (be sure to read the help for this command to properly interpret the results). (Be sure to change the `-c` option to your particular field within your map.txt file).

        compare_categories.py -i final_otu_tables/otu_table.biom --method adonis -i plots/bdiv_otu/weighted_unifrac_dm.txt -m map.txt -c mouse_type -o adonis_ko_vs_wt
        compare_categories.py -i final_otu_tables/otu_table.biom --method anosim -i plots/bdiv_otu/weighted_unifrac_dm.txt -m map.txt -c mouse_type -o anosim_ko_vs_wt

* Compare within vs between b-diversity.

        make_distance_boxplots.py -m map.txt -d plots/bdiv_otu/weighted_unifrac_dm.txt -f mouse_type -o plots/bdiv_box_plots

* Make stacked bar charts.

        summarize_taxa_through_plots.py -i final_otu_tables/otu_table.biom -o plots/taxa_summary
		
		#Note: you can also collapse samples by a category
        summarize_taxa_through_plots.py -i final_otu_tables/otu_table.biom -o plots/taxa_summary -m map.txt -c mouse_type
		
* Compute MD index for IBD.

        compute_taxonomy_ratios.py -i final_otu_tables/otu_table.biom -e md -o map_with_md.txt -m map.txt

* Build and evaluate a classifier using Random Forests (this should only be done for large datasets).

        supervised_learning.py -i otu_table.biom -m map.txt -c mouse_type -o ml -v
		
		#Can also do 5-fold cross validation
		supervised_learning.py -i otu_table.biom -m map.txt -c mouse_type -o ml -v -e cv5

* Normalize OTU table using DESeq2 instead of rarefying, which is an alternative method. Check out the help of this script (normalize_table.py -h) to read more about these methods. 

        normalize_table.py -i clustering/otu_table_high_conf.biom -a DESeq2 -z -o final_otu_tables/otu_table_deseq2_norm_no_negatives.biom