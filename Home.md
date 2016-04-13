Microbiome Helper is a repository that contains several resources to help researchers working with microbial sequencing data: 
* a series of scripts that help process and automate various microbiome and metagenomic bioinformatic tools. 
* workflows for analyzing 16S/18S rRNA and metagenomic data using these tools. 
* tutorials for researchers new to rRNA and/or metagenomic analyses.
* a Virtual Box image that can be used to run our workflows with little or not configuration.

These scripts were produced by the [Integrated Microbiome Resource](http://cgeb-imr.ca/index.html). It is important that you cite the [tools](https://github.com/mlangill/microbiome_helper/wiki/Requirements) that are wrapped by our scripts.

Use the sidebar menu to navigate the wiki.

Before getting started:

* These workflows start with raw paired-end MiSeq data in demultiplexed fastq format.

* Most of the commands allow parallel threads to be used with the -p option. Adjust according to your computer hardware abilities.

* These workflows can be completed in roughly 24 hours on a single quad-core desktop when starting with 25 million paired end reads.