## Data Set

In this tutorial, we are using 189 faecal samples from  [_Host lifestyle affects human microbiota on daily timescales_](http://www.ncbi.nlm.nih.gov/pmc/articles/PMC4405912/) (Lawrence, et al., 2014). These samples were taken from one subject over the course of These samples were taken from one subject over the course of a single year. For the purposes of this live tutorial, only a subset of the sequences from each sample is used.

## Retrieve the Data

    wget <insert final_link here>
    unzip stool_data.zip

The zip file contains two files: `stool_sequences.fasta`, and `map.txt`. Because quality filtering steps are specific to each sequencing platform, we will not address that in this tutorial. The `stool_sequences.fasta` file will contain the sequences from each sample. The `map.txt` is a tab-separated table file that contains the metadata information for each sample.

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

