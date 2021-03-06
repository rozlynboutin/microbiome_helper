These are the answers to the questions from the [Metagenomics Tutorial](https://github.com/mlangill/microbiome_helper/wiki/Metagenomics-Tutorial).  

**Introduction and taxonomic section:**

1. 30
2. 10
3. 60,000/sample
4. MetaPhlAn version 2.5.0 (28 April 2015)
5. s__Neisseria_unclassified	11.8%
6. SRS015044: p__Actinobacteria & SRS015893: p_Firmicutes
7. Phylum Candidatus_Saccharibacteria, present only in sample SRS015044
8. Sample SRS015044 was from supragingival plaque
9. metaphlan_out
10. Yes, there is separation between the tongue samples and the plaque samples but there is no clear separation between subgingival and supragingival. The separation occurs mainly along PCs 1 and 2: ![](https://www.dropbox.com/s/96bistjnv53r3iv/bodysite_PCA_taxonomy_downsample.png?raw=1)

11. You should see this plot:

<img src="https://www.dropbox.com/s/vyko1vrbsyax5z1/taxonomy_f_streptococcaceae_boxplot.png?raw=1" alt="boxplot_answer" width="500" height="500">

**Functional section:**

1. SRS015893
2. No
3. 50 (KOs different across body sites)
4. 2 (KEGG Pathways different across body sites)
5. ko00281: Geraniol degradation . corrected p-value: 4.14e-4
6. 5
7. Higher in the tongue samples.
8. Should look like this:
![](https://www.dropbox.com/s/ibq0saxla5jq5jm/functional_error_corr_downsampled.png?raw=1)