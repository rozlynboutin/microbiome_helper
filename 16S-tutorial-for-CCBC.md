## Data Set

In this tutorial, we are using 189 faecal samples from  [_Host lifestyle affects human microbiota on daily timescales_](http://www.ncbi.nlm.nih.gov/pmc/articles/PMC4405912/) (Lawrence, et al., 2014). These samples were taken from one subject over the course of These samples were taken from one subject over the course of a single year. For the purposes of this live tutorial, only a subset (1,500) of the sequences from each sample is used.

## Retrieve the Data

    wget https://www.dropbox.com/s/kpte51nm17wav9o/stool_data.zip
    unzip stool_data.zip

The zip file contains two files: `stool_sequences.fasta`, and `stool_metadata.csv`. Because quality filtering steps are specific to each sequencing platform, we will not address that in this tutorial. The `stool_sequences.fasta` file will contain the sequences from each sample. The `stool_metadata.csv` is a tab-separated table file that contains the metadata information for each sample.

Let's take a look at the sequences:

    head -n 4 stool_sequences.fasta

<details> 
  <summary>Reveal command output</summary>
<pre><code>
    >Stool262.1259873_0
    TACGGAGGATGCGAGCGTTATCCGGATTTATTGGGTTTAAAGGGAGCGCAGACGGGTCGTTAAGTCAGCTGTGAAAGTTTGGGGCTCAACCTTAAAATTGC
    >Stool262.1259873_1
    TACGTAGGTGGCGAGCGTTATCCGGAATTATTGGGCGTAAAGAGGGAGCAGGCGGCACTAAGGGTCTGTGGTGAAAGATCGAAGCTTAACTTCGGTAAGCC
</code></pre></details>

The FASTA file has labels in the format ">SampleName_SequenceNumber".

## Clustering Sequences

<details> 
  <summary>Reveal all code for this section for easier copy & paste</summary>
<pre><code>
mkdir cluster
vsearch -derep_fulllength stool_sequences.fasta -output cluster/unique.fasta -sizeout -minseqlength 50
vsearch -uchime_denovo cluster/unique.fasta --nonchimeras cluster/nochimeras.fasta
vsearch -sortbysize cluster/nochimeras.fasta -output cluster/sorted.fasta -minsize 2
vsearch -cluster_smallmem cluster/sorted.fasta --id 0.97 --consout cluster/rep_set.fasta --usersort
awk 'BEGIN{OFS="";ORS="";count=0}{if ($0~/>/){if (NR>1) {print "\n"} print ">" count "\n"; count+=1;} else {print $0;}}' cluster/rep_set.fasta > cluster/rep_set_relabel.fasta
vsearch -usearch_global stool_sequences.fasta -db cluster/rep_set_relabel.fasta -strand both -id 0.97 -uc cluster/map.uc -threads 2
wget https://github.com/neufeld/MESaS/raw/master/scripts/mesas-uc2clust
python mesas-uc2clust cluster/map.uc cluster/seq_otus.txt
</code></pre></details>

For this tutorial, we will be using the open-source *vsearch* program to cluster sequences into operational taxonomic units (OTUs). This is a very similar approach to the UPARSE pipeline. There are many other alternative clustering methods, such as open reference clustering.

First, make a new directory to put our results and intermediate files into:

    mkdir cluster

The sequences are reduced to only unique sequences ("dereplicated") to improve computation time:

    vsearch -derep_fulllength stool_sequences.fasta -output cluster/unique.fasta -sizeout -minseqlength 50

Chimeras are detected in a *de novo* fashion, and excluded:

    vsearch -uchime_denovo cluster/unique.fasta --nonchimeras cluster/nochimeras.fasta

Sequences are sorted by size, and singletons are removed (they are mapped back on later):

    vsearch -sortbysize cluster/nochimeras.fasta -output cluster/sorted.fasta -minsize 2

Operational taxonomic units are clustered at 97% sequence identity, and the consensus sequences for the OTUs are output to a FASTA file:

    vsearch -cluster_smallmem cluster/sorted.fasta --id 0.97 --consout cluster/rep_set.fasta --usersort

The OTU names contain a wealth of information that is not required for downstream processing. To reduce complications, we can simplify the names with *awk*:

    awk 'BEGIN{OFS="";ORS="";count=0}{if ($0~/>/){if (NR>1) {print "\n"} print ">" count "\n"; count+=1;} else {print $0;}}' cluster/rep_set.fasta > cluster/rep_set_relabel.fasta

The original sequence file is mapped back onto the OTU consensus sequences:

    vsearch -usearch_global stool_sequences.fasta -db cluster/rep_set_relabel.fasta -strand both -id 0.97 -uc cluster/map.uc -threads 2

Download a helpful Python script that converts vsearch .uc files into QIIME-compatible format:

    wget https://github.com/neufeld/MESaS/raw/master/scripts/mesas-uc2clust

Run this script on our vsearch .uc file:

    python mesas-uc2clust cluster/map.uc cluster/seq_otus.txt

## Taxonomic Classification

We will use sortmerna to assign taxonomic classifications to each representative sequence:

    assign_taxonomy.py -i cluster/rep_set_relabel.fasta -o assigned_taxonomy -m sortmerna

**Note:** If you are running this with <6GB of RAM, *mothur* may be a faster option. It must be installed before using:

    sudo apt-get install mothur
    assign_taxonomy.py -i cluster/rep_set_relabel.fasta -o assigned_taxonomy -m mothur

The *print_qiime_config.py* command will return information about QIIME's set-up. This includes the default taxonomy reference sets. Since we did not specify them in our assign_taxonomy.py commands, the defaults were used. What are the default files?

## Phylogenetic Tree Construction

First, the sequences must be aligned. This can be accomplished using PyNast via QIIME:

    align_seqs.py -i cluster/rep_set_relabel.fasta -o aligned -m pynast

Next, we build the tree using FastTree:

    make_phylogeny.py -i aligned/rep_set_relabel_aligned.fasta -o rep_set.tre -t fasttree

## OTU Table Creation

QIIME will take the OTU map and taxonomic classifications and create an OTU table:

    make_otu_table.py -i cluster/seq_otus.txt -t assigned_taxonomy/rep_set_relabel_tax_assignments.txt -o otu_table.biom

We can use the BIOM toolkit to retrieve OTU table statistics:

    biom summarize-table -i otu_table.biom -o summary.txt

Now we evenly subsampled (rarefy) the OTU table so that each sample has the same number of sequences:

    single_rarefaction.py -i otu_table.biom -d 803 -o rare_otu_table.biom

The BIOM format is a file type called HDF5. This is not a human-readable format. You can convert these files to tab-separated spreadsheets that can be opened with Excel or LibreOffice Calc. This can be very useful for importing your OTU table using languages other than Python, such as R:

    biom convert -i otu_table.biom -o otu_table.tsv --to-tsv --header-key taxonomy

## Downstream Analyses

### Heatmap

OTU table heatmaps can be a great way to quickly glance structure in your OTU table.

    make_otu_heatmap.py -i rare_otu_table.biom -o heatmap.pdf

### Principal Co-ordinates Plots

QIIME will easily handle creation of principal co-ordinates (PCoA):

    beta_diversity_through_plots.py -i rare_otu_table.biom -m stool_metadata.csv -t rep_set.tre -o beta_plots

Investigate the "DAY" and "PHASE" metadata categories. Compare the weighted and unweighted UniFrac results.

### Alpha Rarefaction Plots

Alpha rarefaction plots will compare the alpha diversity measures (number of OTUs, phylogenetic diversity, Shannon, etc.) as a function of the number of sequences. They can indicate whether a sufficient sequence depth has been achieved.

    alpha_rarefaction.py -i otu_table.biom -m stool_metadata.csv -t rep_set.tre -o alpha_plots

### Comparing Groups of Samples

We can compare groups of samples by computing differences in their beta diversity metrics and using statistical methods to assess the significance of these differences. Here we use ANOSIM, a non-parametric method that compares groups of samples by using the rank-order of beta diversity measures.

    compare_categories.py -i plots/unweighted_unifrac_dm.txt --method anosim -m stool_metadata.csv -c PHASE -o anosim_unweighted_unifrac
    compare_categories.py -i plots/weighted_unifrac_dm.txt --method anosim -m stool_metadata.csv -c PHASE -o anosim_weighted_unifrac

Both results return as significant with *p*<0.001. Which has the largest effect size? Does this match your intuition from the PCoA plots?