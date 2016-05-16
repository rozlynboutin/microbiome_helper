## Data Set

In this tutorial, we are using 189 faecal samples from  [_Host lifestyle affects human microbiota on daily timescales_](http://www.ncbi.nlm.nih.gov/pmc/articles/PMC4405912/) (David *et al.*, 2014). These samples were taken from one subject over the course of These samples were taken from one subject over the course of a single year. For the purposes of this live tutorial, only a subset (1,500) of the sequences from each sample is used.

## Retrieve the Data

    wget https://www.dropbox.com/s/kpte51nm17wav9o/stool_data.zip
    unzip stool_data.zip

The zip file contains two files: __`stool_sequences.fasta`__, and __`stool_metadata.csv`__. Because quality filtering steps are specific to each sequencing platform, we will not address that in this tutorial. The `stool_sequences.fasta` file will contain the sequences from each sample. The `stool_metadata.csv` is a tab-separated table file that contains the metadata information for each sample.

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
sudo apt-get install vsearch
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

Install vsearch:

    sudo apt-get install vsearch

The sequences are reduced to only unique sequences ("dereplicated") to improve computation time:

    vsearch -derep_fulllength stool_sequences.fasta -output cluster/unique.fasta -sizeout -minseqlength 50

__`cluster/unique.fasta`__: A FASTA file containing only the unique sequences.

<details> 
  <summary>Reveal command output</summary>
<pre><code>
    vsearch v1.1.3_linux_x86_64, 2.5GB RAM, 4 cores
    https://github.com/torognes/vsearch
    Reading file stool_sequences.fasta 100%  
    28205462 nt in 283495 seqs, min 77, max 101, avg 99
    WARNING: 1 sequences shorter than 50 nucleotides discarded.
    Indexing sequences 100%  
    Dereplicating 100%  
    Sorting 100%
    37029 unique sequences, avg cluster 7.7, median 1, max 40958
    Writing output file 100%  
</code></pre></details>

Chimeras are detected in a *de novo* fashion, and excluded:

    vsearch -uchime_denovo cluster/unique.fasta --nonchimeras cluster/nochimeras.fasta

__`cluster/nochimeras.fasta`__: FASTA file of the unique sequences with chimeric sequences removed.

How many of the sequences were determined to be chimeric?

<details> 
  <summary>Reveal command output</summary>
<pre><code>
    vsearch v1.1.3_linux_x86_64, 2.5GB RAM, 4 cores
    https://github.com/torognes/vsearch
    Reading file cluster/unique.fasta 100%  
    3549373 nt in 37029 seqs, min 77, max 101, avg 96
    Indexing sequences 100%  
    Sorting by abundance 100%
    Counting unique k-mers 100%  
    Detecting chimeras 100%  
    Found 4067 (11.0%) chimeras, 32863 (88.7%) non-chimeras,
    and 99 (0.3%) suspicious candidates in 37029 sequences.
</code></pre></details>

Sequences are sorted by size, and singletons are removed (they are mapped back on later):

    vsearch -sortbysize cluster/nochimeras.fasta -output cluster/sorted.fasta -minsize 2

__`cluster/sorted.fasta`__: FASTA file with the sequences sorted by abundance (highest first), and singletons removed (-minsize 2).

<details> 
  <summary>Reveal command output</summary>
<pre><code>
    vsearch v1.1.3_linux_x86_64, 2.5GB RAM, 4 cores
    https://github.com/torognes/vsearch
    Reading file cluster/nochimeras.fasta 100%  
    3141609 nt in 32863 seqs, min 77, max 101, avg 96
    Indexing sequences 100%  
    Getting sizes 100%  
    Sorting 100%
    Median abundance: 3
    Writing output 100%  
</code></pre></details>

Operational taxonomic units are clustered at 97% sequence identity, and the consensus sequences for the OTUs are output to a FASTA file:

    vsearch -cluster_smallmem cluster/sorted.fasta --id 0.97 --consout cluster/rep_set.fasta --usersort

__`cluster/rep_set.fasta`__: Consensus sequences for each 97% OTU

How many OTUs were created?

<details> 
  <summary>Reveal command output</summary>
<pre><code>
    vsearch v1.1.3_linux_x86_64, 2.5GB RAM, 4 cores
    https://github.com/torognes/vsearch
    Reading file cluster/sorted.fasta 100%  
    703761 nt in 7214 seqs, min 77, max 101, avg 98
    Indexing sequences 100%  
    Masking 100%
    Counting unique k-mers 100%  
    Clustering 100%  
    Writing clusters 100%  
    Clusters: 602 Size min 1, max 450, avg 12.0
    Singletons: 221, 3.1% of seqs, 36.7% of clusters
    Multiple alignments 100%  
</code></pre></details>

The OTU names contain a wealth of information that is not required for downstream processing. To reduce complications, we can simplify the names with *awk*:

    awk 'BEGIN{OFS="";ORS="";count=0}{if ($0~/>/){if (NR>1) {print "\n"} print ">" count "\n"; count+=1;} else {print $0;}}' cluster/rep_set.fasta > cluster/rep_set_relabel.fasta

The original sequence file is mapped back onto the OTU consensus sequences:

    vsearch -usearch_global stool_sequences.fasta -db cluster/rep_set_relabel.fasta -strand both -id 0.97 -uc cluster/map.uc -threads 2

__`cluster/map.uc`__: VSEARCH-formatted mapping file (sequence search hit table)

What proportion of the sequences mapped onto the OTU consensus reads?

<details> 
  <summary>Reveal command output</summary>
<pre><code>
    vsearch v1.1.3_linux_x86_64, 2.5GB RAM, 4 cores
    https://github.com/torognes/vsearch
    Reading file cluster/rep_set_relabel.fasta 100%  
    59453 nt in 602 seqs, min 77, max 102, avg 99
    Indexing sequences 100%  
    Masking 100%
    Counting unique k-mers 100%  
    Creating index of unique k-mers 100%  
    Searching 100%  
    Matching query sequences: 270602 of 283496 (95.45%)
</code></pre></details>

Download a helpful Python script that converts vsearch .uc files into QIIME-compatible format:

    wget https://github.com/neufeld/MESaS/raw/master/scripts/mesas-uc2clust

Run this script on our vsearch .uc file:

    python mesas-uc2clust cluster/map.uc cluster/seq_otus.txt

__`cluster/seq_otus.txt`__: Mapping file that relates sequences to the OTU that they belong to.

## Taxonomic Classification

Mothur is a 16S analysis software toolkit, much like QIIME. However, it has its own taxonomic classification utility that can be accessed through QIIME's `assign_taxonomy.py` command. Mothur's implementation is a naive Bayesian approach, similar to the popular Ribosomal Database Project (RDP) classifier. It must be installed before using:

    sudo apt-get install mothur
    assign_taxonomy.py -i cluster/rep_set_relabel.fasta -o assigned_taxonomy -m mothur

__`assigned_taxonomy/rep_set_relabel_tax_assignments.txt`__: Taxonomic assignments for each OTU, along with posterior probability (confidence) value.

The `print_qiime_config.py` command will return information about QIIME's set-up. This includes the default taxonomy reference sets. Since we did not specify them in our `assign_taxonomy.py` commands, the defaults were used. What are the default files?

## Phylogenetic Tree Construction

First, the sequences must be aligned. This can be accomplished using PyNast via QIIME:

    align_seqs.py -i cluster/rep_set_relabel.fasta -o aligned -m pynast

__`rep_set_relabel_aligned.fasta`__: FASTA file containing the aligned OTU consensus sequences.

Next, we build the tree using FastTree:

    make_phylogeny.py -i aligned/rep_set_relabel_aligned.fasta -o rep_set.tre -t fasttree

__`rep_set.tre`__: Phylogenetic tree in Newick format.

## OTU Table Creation

QIIME will take the OTU map and taxonomic classifications and create an OTU table:

    make_otu_table.py -i cluster/seq_otus.txt -t assigned_taxonomy/rep_set_relabel_tax_assignments.txt -o otu_table.biom

__`otu_table.biom`__: BIOM-formatted OTU table. This is a binary file and is not human readable.

We can use the BIOM toolkit to retrieve OTU table statistics:

    biom summarize-table -i otu_table.biom -o summary.txt

What is the lowest sample sequence depth?

<details> 
  <summary>Reveal table summary output</summary>
<pre><code>
    Num samples: 189
    Num observations: 602
    Total count: 270602
    Table density (fraction of non-zero values): 0.187
    Counts/sample summary:
    Min: 803.0
    Max: 1488.0
    Median: 1445.000
    Mean: 1431.757
    Std. dev.: 74.096
    Sample Metadata Categories: None provided
    Observation Metadata Categories: taxonomy
    Counts/sample detail:
    Stool441.1259692: 803.0
    Stool382.1260123: 815.0
    Stool386.1260377: 1076.0
    Stool444.1260382: 1299.0
    Stool535.1259920: 1320.0
    ...
</code></pre></details>

Now we evenly subsampled (rarefy) the OTU table so that each sample has the same number of sequences:

    single_rarefaction.py -i otu_table.biom -d 803 -o rare_otu_table.biom

__`rare_otu_table.biom`__: Rarefied (evenly subsampled) OTU table in BIOM format.

The BIOM format is a file type called HDF5. This is not a human-readable format. You can convert these files to tab-separated spreadsheets that can be opened with Excel or LibreOffice Calc. This can be very useful for importing your OTU table using languages other than Python, such as R:

    biom convert -i otu_table.biom -o otu_table.tsv --to-tsv --header-key taxonomy

__`otu_table.tsv`__: Tab-separated OTU table that is human-readable and can be opened with a spreadsheet viewer (given it isn't too large to fit in memory!)

## Downstream Analyses

### Heatmap

OTU table heatmaps can be a great way to quickly glance structure in your OTU table.

    make_otu_heatmap.py -i rare_otu_table.biom -o heatmap.pdf

__`heatmap.pdf`__: PDF of the heatmap figure, with OTUs as rows and samples as columns.

<details> 
  <summary>Reveal sample plot output</summary>
<pre><code>
![](http://imgur.com/NAvHU9g.png)
</code></pre></details>

### Principal Co-ordinates Plots

QIIME will easily handle creation of principal co-ordinates (PCoA):

    beta_diversity_through_plots.py -i rare_otu_table.biom -m stool_metadata.csv -t rep_set.tre -o beta_plots

__`beta_plots/weighted_unifrac_emperor_pcoa_plot/index.html`__: Webpage that can be opened to view interactive weighted UniFrac PCoA plots.

__`beta_plots/unweighted_unifrac_emperor_pcoa_plot/index.html`__: Webpage that can be opened to view interactive unweighted UniFrac PCoA plots.

__`beta_plots/weighted_unifrac_dm.txt`__: Tab-separated file containing the samples as rows and columns, and their weighted UniFrac beta diversity distance measures.

__`beta_plots/unweighted_unifrac_dm.txt`__: Tab-separated file containing the samples as rows and columns, and their unweighted UniFrac beta diversity distance measures.

Investigate the "DAY" and "PHASE" metadata categories. Compare the weighted and unweighted UniFrac results. Which one of these measures separates the samples by PHASE better?

<details> 
  <summary>Reveal sample weighted UniFrac plot output</summary>
<pre><code>
![](http://imgur.com/BFvKnbc.png)
Coloured by PHASE.
</code></pre></details>

<details> 
  <summary>Reveal sample unweighted UniFrac plot output</summary>
<pre><code>
![](http://imgur.com/HNa2kC2.png)
Coloured by PHASE.
</code></pre></details>

### Alpha Rarefaction Plots

Alpha rarefaction plots will compare the alpha diversity measures (number of OTUs, phylogenetic diversity, Shannon, etc.) as a function of the number of sequences. They can indicate whether a sufficient sequence depth has been achieved.

    alpha_rarefaction.py -i otu_table.biom -m stool_metadata.csv -t rep_set.tre -o alpha_plots

__`alpha_plots/alpha_rarefaction_plots/rarefaction_plots.html`__: Webpage for interactive viewing of alpha rarefaction plots.

Is there a difference between the alpha diversity metrics across any of the phases?

<details> 
  <summary>Reveal sample alpha rarefaction plot output</summary>
<pre><code>
![](http://imgur.com/GL4Lo6b.png)
![](http://imgur.com/qPFwVeB.png)
Coloured by PHASE.
</code></pre></details>

### Comparing Groups of Samples

We can compare groups of samples by computing differences in their beta diversity metrics and using statistical methods to assess the significance of these differences. Here we use ANOSIM, a non-parametric method that compares groups of samples by using the rank-order of beta diversity measures.

    compare_categories.py -i beta_plots/unweighted_unifrac_dm.txt --method anosim -m stool_metadata.csv -c PHASE -o anosim_unweighted_unifrac
    compare_categories.py -i beta_plots/weighted_unifrac_dm.txt --method anosim -m stool_metadata.csv -c PHASE -o anosim_weighted_unifrac

__`anosim_weighted_unifrac/anosim_results.txt`__: Weighted UniFrac ANOSIM results comparing PHASE metadata groups.

__`anosim_unweighted_unifrac/anosim_results.txt`__: Unweighted UniFrac ANOSIM results comparing PHASE metadata groups.

<details> 
  <summary>Reveal ANOSIM output</summary>
<pre><code>
=== anosim_weighted_unifrac/anosim_results.txt ===
method name     ANOSIM
test statistic name     R
sample size     189
number of groups        3
test statistic  0.24827535059673073
p-value 0.001
number of permutations  999
=== anosim_unweighted_unifrac/anosim_results.txt ===
method name     ANOSIM
test statistic name     R
sample size     189
number of groups        3
test statistic  0.70407488355700698
p-value 0.001
number of permutations  999
</pre></code></details>

Both results return as significant with *p*<0.001. Which has the largest effect size? Does this match your intuition from the PCoA plots?