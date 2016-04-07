This wiki is **currently under construction**. For the microbiome helper pipeline please see the main [GitHub page](https://github.com/mlangill/microbiome_helper).

Microbiome helper is a series of scripts that help process and automate various microbiome and metagenomic bioinformatic tools. The main workflows are for analyzing **16S** and **metagenomic** data.

These scripts were produced as part of the [Integrated Microbiome Resource](http://cgeb-imr.ca/index.html). It is important that you **cite** the [tools](https://github.com/mlangill/microbiome_helper/wiki/Requirements) that are wrapped by our scripts.

If you're looking for the MicrobiomeHelper Virtual Box Image, it can be found [here](https://github.com/mlangill/microbiome_helper/wiki/MicrobiomeHelper-Virtual-Box-image).

### Before getting started:

* These workflows starts with raw paired-end MiSeq data in demultiplexed fastq format.

* Most of the commands allow parallel threads to be used with the -p option. Adjust according to your computer hardware abilities.

* These workflows can be completed in roughly 24 hours on a single quad-core desktop when starting with 25 million paired end reads.